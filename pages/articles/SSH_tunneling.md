# SSH Tunneling

SSH tunneling is a method to create an encrypted tunnel to a remote machine to forward a port or to get round connection restrictions.

## Local port forwarding
Local port forwarding will open a port on the local machine that will forward all traffic to a remote machine via a gateway.

```bash
ssh -L ([local_IPaddr]):[local_port]:[remote_machine]:[remote_port] [gateway]

Optional: -nNT to not open a tty.
```

Example usage: You want to forward your local port 9000 to be able to access a web page on your internal network with the address 192.168.100.116, using a SSH connection to an internal machine(Internet facing) with the address 198.51.100.123.

```bash
ssh -L 9000:192.168.100.116:80 han_solo@198.51.100.123
```

## Remote port forwarding
Remote port forwarding will open a port on the remote machine that will forward all traffic to a certain machine and port.

```bash
ssh -R ([permitted_IPaddr]):[remote_port]:[needed_machine]:[needed_port] [remote_machine]

Optional: -nNT to not open a tty.
```
