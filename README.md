# LANcache
Dynamically Cache Steam Installs using [Nginx](http://nginx.org/)

# Requirements
Due to the features used in this configuration it requires
* [nginx 1.6.0+](http://nginx.org/).

# Installation

Create a FreeBSD jail to run as the cache server. Attach a zvol in your jail as /data. How you manage this part is up to you.

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

