# Wireguard install

## Server setup

### Server interface configuration

To install Wireguard and the CLI utilities follow to instructions for your OS from the official website: <https://www.wireguard.com/install/>

The next step is to generate the keys:

```bash
cd /etc/wireguard
umask 077
wg genkey | sudo tee privatekey | wg pubkey | sudo tee publickey
```

After this you have to configure the Wireguard interface, I will call it `wg0`. Create the `/etc/wireguard/wg0.conf` file and add the following:
```
[Interface]
PrivateKey = <your server private key here>
Address = 10.10.0.1/24
Address = fd86:ea04:1111::1/64
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
ListenPort = 51820
```

The Address field represents the address of the VPN interface, all connections should be in that network.

To activate routing add the following lines to ''/etc/sysctl.conf'' and then run the ''sysctl -p'' command.
```
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Now you can bring up the VPN interface with `wg-quick up wg0`.

### Adding VPN peers

After creating clients (next section) run the following command to add them to the server:
```bash
wg set wg0 peer <client-public-key> allowed-ips <client ip address and mask>
```

You can see your status with `wg`.

To start Wireguard at system startup, enable it in systemctl:

```
systemctl enable wg-quick@wg0
```

## Client setup

### Android

For the client I will use the Android app.

After downloading the app click on the blue plus sign and select Create from scratch. here add the following info:

```
#Interface
Name: wg0
Private key: [Click GENERATE]
Public key: [Click GENERATE]
Address: <client's address, same network as the server>
DNS servers: 1.1.1.1

#Peer
Public key: <server's public key>
Allowed IPs: <networks you want to reach from the phone (preferably VPN network), or 0.0.0.0/0 for all>
Endpoint: <server's address and port>
Persistent keepalive: <25 if you want to keep the connections inside NAT>
```

### Linux desktop

Install `wireguard` from your distro's package manager (<https://www.wireguard.com/install/>).

Firstly generate the key pair:

```bash
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

Now you need to create the peer configuration file for the interface (`/etc/wireguard/wg0.conf`):

```
[Interface]
PrivateKey = <key from /etc/wireguard/public.key>
Address = <client's address, same network as the server>

[Peer]
PublicKey = <server's public key>
AllowedIPs = <networks you want to reach from the phone (preferably VPN network), or 0.0.0.0/0 for all>
Endpoint = <server's address and port>
```

Now go back to the `Adding VPN peers` in the Server section to add the newly created peer.

To connect to the VPN use: `sudo wg-quick up wg0`. `sudo wg-quick down wg0` to disconnect.