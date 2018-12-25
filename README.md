# SteamCache Jail
Dynamically Cache Steam Installs using [Nginx](http://nginx.org/)

# Requirements
Due to the features used in this configuration it requires
* [nginx 1.6.0+](http://nginx.org/).

# Installation

## Jail Creation

You can create this jail any how you want. For this readme though we will be using FreeNAS.

First create a new jail for the nginx cache server.

![jail creation](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/create_jail.png)

Next we want to stop the jail you can find the stop button inside the jails sections of
FreeNAS. This step is important, because we can't mount the dataset if the jail is
running.

Next we want to add a new databaset for the /data directory. This dataset will hold all
the cached steam installs.

![add dataset](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/add_dataset.png)

Next configure the dataset.

![create dataset](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/create_dataset.png)

The last step is to add the mountpoint to the jail.

![add_mountpoint](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/add_mountpoint.png)

Now we have our jail up and running you can either configure ssh or use the built in
shell option for connect to your jail.


## Jail Commannds

```
$ pkg install nginx
$ cd /usr/local/etc
$ mv nginx nginx_original
$ git clone https://github.com/gregf/steamcache-jail nginx
$ sysrc nginx_enable="YES"
```

Edit your /etc/hosts file on the jail and create the following entries

192.168.1.35 lancache-steam

Replace the IP used here with the IP of your jail.

```
$ mkdir -p /data/cache/www
$ chown -R nobody:root /data/cache/www
$ service nginx start
```

# DNS Changes

Fetch the generate script from any FreeBSD machine and run the script to get the most up to date DNS entries. These point the steam cache servers to our local proxy server we created above.

```
$ mkdir ~/dns
$ cd ~/dns
$ fetch https://raw.githubusercontent.com/gregf/steamcache-jail/master/scripts/generate.sh
```

Edit ~/dns/.env to contain your jails IP. Example below.

```
LANCACHE_IP="192.168.1.35"
```

Now run the generate script.

```
$ ./generate.sh
```

If you are using PfSense this should be straight forward you just paste the output into the following section.

* Services > DNS Resolver > General Settings > Custom Options


# Finishing up

After this when you download anything from steam it should be cached to your jail. As a test you can download any game and let it get cached, uninstall it, then download it again. The 2nd time it
should come from your LAN and be much faster. This all depends on your network connection, and how fast your drives are. I would recommend using a SSD if possible for your cache.

