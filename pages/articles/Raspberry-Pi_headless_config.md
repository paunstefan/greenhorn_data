# Raspberry-Pi Headless config

## UPDATE AS OF APRIL 2022

The SSH, WiFi and user creation for headless installs are done only using the `rpi-imager` software, as the default `pi` user has been removed and
there is no alternative way to configure a user without a GUI or a direct terminal connection.

## SSH (obsolete)

To enable SSH, go to the `boot` partition on the SD card and create an empty `ssh` file.

## Wifi configuration (obsolete)

Go to the same `boot` partition and create an `wpa_supplicant.conf` in which you will write the following:

```
country=US # Your 2-digit country code
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="YOUR_NETWORK_NAME"
    psk="YOUR_PASSWORD"
}
```

The Pi will automatically connect to the configured network, you just need to look for its IP address.

## User creation (obsolete)

Now create a new user and add it to the `sudo` group:

```
adduser [username]
usermod -aG sudo [username]

# you can also delete the default user
sudo userdel -f pi
```

## Static IP address

To assign a static IP address you need to edit the `/etc/dhcpcd.conf` file. Add the following:

```
interface [intf]
static ip_address=STATIC_IP/24
static routers=ROUTER_IP
static domain_name_servers=DNS_IP
```

## Enable UART

To enable UART/serial use the `raspi-config` utility. Under the `Interface options` you can enable serial and disable the console on it if you want.