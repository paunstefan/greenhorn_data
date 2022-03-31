# Rust Raspberry Pi Pico

Setup guide for running code on the Raspberry Pi Pico. Apllicable to other embedded targets too (with small changes).

## Minimum runnable project

After creating a new Rust project as you normally would, add a `.cargo/config.toml` file where you will add the target build config.

```toml
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "elf2uf2-rs -d"
#runner = "probe-run --chip RP2040"

rustflags = [
  # This is needed if your flash or ram addresses are not aligned to 0x10000 in memory.x
  # See https://github.com/rust-embedded/cortex-m-quickstart/pull/95
  "-C", "link-arg=--nmagic",

  # LLD (shipped with the Rust toolchain) is used as the default linker
  "-C", "link-arg=-Tlink.x",

  # Code-size optimizations.
  #   trap unreachable can save a lot of space, but requires nightly compiler.
  #   uncomment the next line if you wish to enable it
  # "-Z", "trap-unreachable=no",
  "-C", "inline-threshold=5",
  "-C", "no-vectorize-loops",
]


[build]
target = "thumbv6m-none-eabi"
```

Also make sure you have the `thumbv6m-none-eabi` rustc target installed.

To be able to run the code on the Pico, there is also a need to give Rust a linker script. Create a `memory.x` file with the following contents:

```
MEMORY {
    BOOT2 : ORIGIN = 0x10000000, LENGTH = 0x100
    FLASH : ORIGIN = 0x10000100, LENGTH = 2048K - 0x100
    RAM   : ORIGIN = 0x20000000, LENGTH = 256K
}

EXTERN(BOOT2_FIRMWARE)

SECTIONS {
    /* ### Boot loader */
    .boot2 ORIGIN(BOOT2) :
    {
        KEEP(*(.boot2));
    } > BOOT2
} INSERT BEFORE .text;
```

This will be specific to every board.

So Rust can always find it, add a `build.rs` file:

```rust
//! This build script copies the `memory.x` file from the crate root into
//! a directory where the linker can always find it at build time.
//! For many projects this is optional, as the linker always searches the
//! project root directory -- wherever `Cargo.toml` is. However, if you
//! are using a workspace or have a more complicated build setup, this
//! build script becomes required. Additionally, by requesting that
//! Cargo re-run the build script whenever `memory.x` is changed,
//! updating `memory.x` ensures a rebuild of the application with the
//! new memory settings.

use std::env;
use std::fs::File;
use std::io::Write;
use std::path::PathBuf;

fn main() {
    // Put `memory.x` in our output directory and ensure it's
    // on the linker search path.
    let out = &PathBuf::from(env::var_os("OUT_DIR").unwrap());
    File::create(out.join("memory.x"))
        .unwrap()
        .write_all(include_bytes!("memory.x"))
        .unwrap();
    println!("cargo:rustc-link-search={}", out.display());

    // By default, Cargo will re-run a build script whenever
    // any file in the project changes. By specifying `memory.x`
    // here, we ensure the build script is only re-run when
    // `memory.x` is changed.
    println!("cargo:rerun-if-changed=memory.x");
}
```

In the `Cargo.toml` file, add the following dependencies:

```toml
cortex-m = "0.7.4"
cortex-m-rt = "0.7.1"
embedded-hal = "0.2.7"
embedded-time = "0.12.1"
panic-halt = "0.2.0"
rp-pico = "0.3.0"
```

Finally, the `main.rs` file should just import some of the dependencies we added.

```rust
#![no_std]
#![no_main]

// The macro for our start-up function
use cortex_m_rt::entry;

// GPIO traits
use embedded_hal::digital::v2::OutputPin;

// Time handling traits
use embedded_time::rate::*;

// Ensure we halt the program on panic (if we don't mention this crate it won't
// be linked)
use panic_halt as _;

// Pull in any important traits
use rp_pico::hal::prelude::*;

// A shorter alias for the Peripheral Access Crate, which provides low-level
// register access
use rp_pico::hal::pac;

// A shorter alias for the Hardware Abstraction Layer, which provides
// higher-level drivers.
use rp_pico::hal;

/// Entry point to our bare-metal application.
///
/// The `#[entry]` macro ensures the Cortex-M start-up code calls this function
/// as soon as all global variables are initialised.
///
/// The function configures the RP2040 peripherals, then blinks the LED in an
/// infinite loop.
#[entry]
fn main() -> ! {
    // You code here
}
```

If you have installed `elf2uf2-rs` and the Pico in plugged into the USB port (while holding the BOOTSEL button), running is done using `cargo run`.


## Sources

* <https://docs.rust-embedded.org/book/intro/index.html>
* <https://github.com/rp-rs/rp-hal>