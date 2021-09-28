# SSH key authentication

## Client side

In terminal use ssh-keygen to create your keys:

```bash
ssh-keygen
```

After you created the needed directories on the server (see below), run the following command to automatically copy the key:

```bash
ssh-copy-id user@host
```

## Server side

After creating the public key on the client machine, create a `.ssh` directory on the server machine, inside which you make an `authorized_keys` file with the public key, on 1 line beginning with `ssh-rsa` OR run the ssh-copy-id command (see above).

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Key example
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDU...
```

Make sure `/etc/ssh/sshd_config` contains:
```
AuthorizedKeysFile %h/.ssh/authorized_keys
```

## Putty
Start PuTTYgen, generate the key and save the public and private key in a safe place.

In the application, under the Data tab set your username. Under the SSH->Auth tab choose your public key. After that save the SSH profile.