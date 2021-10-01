## Make Apt use IPv4

Go the the `/etc/gai.conf` file. Uncomment the following line:

```bash
precedence ::ffff:0:0/96  100
```