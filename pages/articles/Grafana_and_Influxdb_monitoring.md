# Grafana and InlfuxDB monitoring
I will explain how to set up a simple monitoring dashboard using Grafana, InfluxDB and Telegraf, on an Ubuntu system (the installation might be a bit different for other Linux distros).

## What you need

* Grafana: An open source platform for building graphs. It runs in a web environment and can be linked with a number of different databases.
* InfluxDB: The database we will use. It is optimized for time series data, which is what we need for our monitoring needs.
* Telegraf: It's a metric collection daemon that provides the data to our InfluxDB database.

## Installing the software

### Grafana
You need to add the Grafana repository. Create a file `/etc/apt/sources.list.d/grafana.list` and add the following to it.

```
deb https://packages.grafana.com/oss/deb stable main
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Now you can install Grafana and start it:

```
apt install grafana

systemctl daemon-reload
systemctl start grafana-server
systemctl enable grafana-server.service
```

You can access it at [host]:3000, default login is admin:admin.

[Grafana documentation](http://docs.grafana.org/)

### InfluxDB

InfluxDB is already present in the Ubuntu repositories.

```
apt install influxdb python3-influxdb influxdb-client
```

Access the command line interface with `influx`. Here you can create your database and users.

```
CREATE DATABASE grafana_test
CREATE USER admin WITH PASSOWRD 'pass' WITH ALL PRIVILEGES
CREATE USER paunstefan WITH PASSWORD 'pass'
GRANT ALL ON grafana_test TO paunstefan
```

Enable authentication in the [http] section of the configuration file `/etc/influxdb/influxdb.conf`:

```
[http]  
enabled = true  
bind-address = ":8086"
auth-enabled = true
```

[InfluxDB documentation](https://www.docs.influxdata.com/)

### Telegraf

As with Grafana, you need to add the needed repository to install Telegraf.

```
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release

echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt-get update && sudo apt-get install telegraf
```

Now start and enable the service:

```
systemctl daemon-reload
systemctl start telegraf
systemctl enable telegraf
```

To make Telegraf output information to InfluxDB you need to modify the configuration file `/etc/telegraf/telegraf.conf`. Here you will go to the `outputs.influxdb` section.

```
[[outputs.influxdb]]
urls = ["host:8086"]
database = "grafana_test"

## HTTP basic auth
username = "paunstefan"
password = "pass"
```

You can find and configure what data Telegraf will send under the `inputs` sections. It will automatically collect the configured information and send it to the database.

Now you can test and if everything is good, restart Telegraf:

```
telegraf -test

systemctl restart telegraf
```

[Telegraf documentation](https://docs.influxdata.com/telegraf/v1.9/)

## Grafana web configuration

As I said earlier, you can access the Grafana dashboard at `[host]:3000` and the default credentials are admin:admin. 

The first step is adding a Data Source, this will be our InflluxDB database.
![Data Source](/static/images/data_source.png)

After this, go the the plus sign on the left and create a Dashboard, in which you will create a panel.

Here is an example panel for memory usage:

![Dashboard](/static/images/dashboard.png)

Every panel has a high degree of customization and you can find even more options by adding additional plugins. 

Now everything should be up and running, you just need to decide what information you want to see.

### External links
* <https://www.reddit.com/r/homelab/comments/603wur/dashboard_tutorial_grafana_influxdb_and_telegraf>
* <https://www.smarthomeblog.net/openhab-persistence-grafana-dashboard/>
* <https://lkhill.com/telegraf-influx-grafana-network-stats/>
* <https://tracyboggiano.com/archive/2018/04/guide-for-setting-up-telegraf-to-monitor-influxdb/>
