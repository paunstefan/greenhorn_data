# Linux SDR Tutorial

## Setting up the RTL-SDR

Plug the RTL-SDR dongle and use `lsusb` to check if the computer sees it. You should see something like `Realtek Semiconductor Corp. RTL2838 DVB-T`.

Now you need to blacklist the standard DVB drivers and replace them with the RTL-SDR drivers. Edit the `/etc/modprobe.d/blacklist-dvb.conf` file adding:
```bash
blacklist dvb_usb_rtl28xxu
```

Next install the new drivers:
```bash
sudo apt-get install rtl-sdr
```

Restart the computer for the changes to take place.

Enter a terminal and run `rtl_test`, if you see some output you are all set up.

## Install GQRX
It's in the Ubuntu repos, so you only need to run:
```bash
apt install gqrx-sdr

#you may also want to install gnuradio and librtlsdr-dev
```


## References

* [Set up PDF](https://www.nooelec.com/store/downloads/dl/file/id/72/product/0/nesdr_installation_manual_for_ubuntu.pdf)
* [GQRX tutorial](http://gqrx.dk/doc/practical-tricks-and-tips)
* [Nooelec help page](https://www.nooelec.com/store/qs/)
