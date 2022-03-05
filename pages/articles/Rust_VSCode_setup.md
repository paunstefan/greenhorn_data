# Rust VSCode setup

Visual Studio Code is at this moment the best editor/IDE for Rust, you have all the tools you need and very little configuration is needed.

The first step, common to all setups, is to install the `rust-analyzer` and `CodeLLDB` VSCode extensions.

## Native

If you are running natively, pretty much everything will work out of the box. The only thing you might want to do
is create a `launch.json` file for debugging. VSCode can do this automatically, you have to go to the `Run and Debug` tab
and click the `create a launch.json file` button.

## Raspberry Pi cross-compile

### Building

To cross compile for Raspberry Pi (or other ARM based computers) the process is a little more complicated.

First you need to install the Rust targets you need.

```bash
# Supports Pi 0/1
rustup target add arm-unknown-linux-gnueabihf
# Supports Pi 2/3/4
rustup target add armv7-unknown-linux-gnueabihf
```

Then you need to download the GNU toolchain (you'll need the linker for your target platform).
Go to <https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads> and download the AArch32 toolchains.

* RPi 0/1: arm-linux-gnueabihf
* RPi 2/3/4: arm-none-linux-gnueabihf

Extract it and add the `bin` directory to your path. (you can also add it to your `.bashrc` to be permanent)

```bash
export PATH="PATH_TO_TOOLCHAIN_FOLDER/bin:$PATH"
```

Now in your Rust project directory, create a new `.cargo` directory, and a `config` file. Inside, add the needed linker.

```toml
# Pi 0/1
[target.arm-unknown-linux-gnueabihf]
linker = "arm-linux-gnueabihf-gcc"

# Pi 2/3/4
[target.armv7-unknown-linux-gnueabihf]
linker = "arm-none-linux-gnueabihf-gcc"

[build]
# (Optional) Set default target for cargo build
target = "armv7-unknown-linux-gnueabihf"
# rustflags = ["-C", "linker=arm-none-linux-gnueabihf-gcc"]
```

To build your program run:

```bash
cargo build
```

To run on the Pi:

```bash
# upload binary
sshpass -p 'raspberry' scp -r ./target/armv7-unknown-linux-gnueabihf/debug/pi_project pi@$PI_IP:/home/pi

# execute binary
sshpass -p 'raspberry' ssh pi@$PI_IP './pi_project'
```

Using `sshpass` like this is easy, but may not be safe, you should use SSH Key authentication.

To make it all easier, create a `runner.sh` file that will contain a script to automate the whole copying and running on the Pi.
```bash
#!/bin/bash
pi_user=[USERNAME]
pi_addr=[PI_ADDR]
target="armv7-unknown-linux-gnueabihf"
binary_name=[PROGRAM_NAME]

print_usage()
{
 echo "
    Raspberry Pi Rust Deployment Helper
Builds the image and transfers and optionally run it on the target system,
with or without debugging.
Usage:
    $(basename $0) [flags]
Flags:
    -x               Run the image
    -d               Run gdbserver on remote
    -s               Start GDB
    -r               Build release version
Observations:
    Login password goes into the SSHPASS environment variable.
"
}

# Set flags to false
run=0
dbg=0
start=0
version=debug

# Check flags given
while test $# != 0
do
    case "$1" in
    -x) run=1 ;;
    -d) dbg=1 ;;
    -s) start=1 ;;
    -r) version=release ;;
    --) shift; break;;
    *)  print_usage 
        exit 1
        ;;
    esac
    shift
done

binary_path=./target/$target/$version

if [ $dbg = 1 ]; then
    run=0
fi

# Uncomment if you want to see the commands being run
#set -o xtrace

echo "Building program"
if [ $version = debug ]; then
    cargo build --target $target
else
    cargo build --release --target $target
fi

echo "Transfering to Raspberry Pi"
sshpass -p "${SSHPASS}" scp -r $binary_path/$binary_name $pi_user@$pi_addr:/tmp/
echo "Transfer complete"

if [ $run = 1 ]; then
    echo "Running remote program"
    sshpass -p "${SSHPASS}" ssh $pi_user@$pi_addr "/tmp/${binary_name}"
fi

if [ $dbg = 1 ]; then
    echo "Running remote debugger"
    sshpass -p "${SSHPASS}" ssh $pi_user@$pi_addr "killall -q gdbserver; gdbserver *:12345 /tmp/${binary_name}" &
    sleep 2
    if [ $start = 1 ]; then
        echo "Connecting to remote GDB"
        arm-none-linux-gnueabihf-gdb -q -ex "target remote $pi_addr:12345" -ex "break main" -ex "continue" $binary_path/$binary_name
    fi
fi
```

It also contains an option for debugging that launches a remote `gdbserver` to which you will connect.

### Integration with VSCode

To integrate the build and debug process into VSCode, you need to create the 
`launch.json` and `tasks.json` inside de `.vscode` directory.

The `tasks.json` file will define tasks that can be run from within VSCode 
(you can access them by pressing F1). Here we'll add tasks for building, running and debugging the program.

```json
{
    "version": "2.0.0",
    "options": {
        "env": {
            "SSHPASS": [YOUR_PASSWORD]
        }
    },
    "tasks": [
        {
            "label": "rust: cargo build",
            "type": "shell",
            "command": "~/.cargo/bin/cargo",
            "args": [
                "build"
            ],
            "problemMatcher": [
                "$rustc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
        {
            "label": "rust: run remotely",
            "type": "shell",
            "command": "~/.cargo/bin/cargo",
            "args": [
                "run"
            ],
            "problemMatcher": [
                "$rustc"
            ],
            "group": "build",
            "dependsOn": [
                "rust: cargo build",
            ],
            "linux": {
                "command": "${workspaceFolder}/runner.sh",
                "args": [
                    "-x"
                ],
            },
        },
        {
            "label": "rust: remote dbg with server",
            "type": "shell",
            "command": "${workspaceFolder}/runner.sh",
            "args": [
                "-d",
            ],
            "group": "none",
            "dependsOn": [
                "rust: cargo build",
            ],
            "linux": {
                "command": "${workspaceFolder}/runner.sh",
                "args": [
                    "-d"
                ],
            },
            "isBackground": true,
            "problemMatcher": {
                "pattern": [
                    {
                        "regexp": ".",
                        "file": 1,
                        "location": 2,
                        "message": 3
                    },
                ],
                "background": {
                    "activeOnStart": true,
                    "beginsPattern": "^.*Listening on port.*$",
                    "endsPattern": "^Child exited with status.*$"
                }
            }
        },
    ]
}
```

It mostly just calls the `runner.sh` script.

The `launch.json` file defines options for debugging a program.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "custom",
            "name": "Remote Debugging With Server",
            "preLaunchTask": "rust: remote dbg with server",
            "targetCreateCommands": [
                "target create ${workspaceFolder}/target/armv7-unknown-linux-gnueabihf/debug/[PROGRAM_NAME]"
            ],
            "processCreateCommands": [
                "gdb-remote [PI_ADDR]:12345"
            ]
        },
    ]
}
```

Now if you use `Start Debugging` with the `Remote Debugging With Server` the debugger will connect to the remote target. You can add breakpoints and step through the code as if it was running locally.