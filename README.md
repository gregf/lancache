# SteamCache Jail
Dynamically Cache Steam Installs using [Nginx](http://nginx.org/)

# Requirements
Due to the features used in this configuration, **[nginx 1.6.0+](http://nginx.org/).**
is required.

# Table of Contents

1. <a href="#jail-creation">Jail Creation</a>
2. <a href="#jail-commands">Jail Commands</a>
3. <a href="#dns-changes">DNS Changes</a>
4. <a href="#finishing-up">Finishing Up</a>

# Installation

## Jail Creation

A FreeBSD jail is required. How the jail is created doesn't matter. This
README will use [FreeNAS](https://www.freenas.org/) to setup and
configure the jail.

Create a new jail for the nginx cache server.

![jail creation](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/create_jail.png)

The Jail Wizard in FreeNAS can quickly create a new jail. Stop the newly
created jail until configuration is complete.

Create a new dataset to use for the /data directory. This dataset will
store the cached steam installation files.

![add dataset](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/add_dataset.png)

Configure the dataset.

![create dataset](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/create_dataset.png)

Add the dataset as a mountpoint to the jail.

![add_mountpoint](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/add_mountpoint.png)

Start the jail. Now that the jail configuration is complete, the jail can
be started. In FreeNAS select the jail, then click the play button, or
click the Options icon -> Start.

_The next steps must be done in the new jail. Configure SSH access, or use
the built in Shell option, to connect to the running jail._

## Jail Commands

```
$ pkg install nginx-full
$ cd /usr/local/etc
$ mv nginx nginx_original
$ git clone https://github.com/gregf/steamcache-jail nginx
$ sysrc nginx_enable="YES"
```

Edit the `/etc/hosts` file on the jail to create the following entry:

`JAIL_IP lancache-steam`

Replace **JAIL_IP** with the IP used for the new jail.

```
$ mkdir -p /data/cache/www
$ chown -R nobody:nobody /data/cache/www
$ service nginx start
```

# DNS Changes

Fetch the generate script from any FreeBSD machine then run the script to
get the most up to date DNS entries. The entries created point the steam
cache servers to the local proxy server that was just created.

```
$ mkdir ~/dns
$ cd ~/dns
$ fetch https://raw.githubusercontent.com/gregf/steamcache-jail/master/scripts/generate.sh
```

Edit `~/dns/.env` to update the `LANCACHE_IP` variable with the IP of the
jail to use.

```
LANCACHE_IP="JAIL_IP"
```

Replace **JAIL_IP** with the IP used for the new jail.

Run the generate script.

```
$ ./generate.sh
```

## Using PfSense
Copy the output from generate.sh.
Go to _Services > DNS Resolver > General Settings > Custom Options_

Paste the output in the Custom options text box.

![dns changes](https://raw.githubusercontent.com/gregf/steamcache-jail/master/imgs/dns.png)

# Finishing up

That's it! Now anything downloaded from steam will be cached to this jail.
To test, download a game, let it cache, then uninstall the game. Download
the same game again. The game should now come from the cache saved in the
jail over the LAN connection and be noticeably faster. The speed depends
on the local network connection, and how fast the drive the data is
stored on, but the cached copy should be much faster than downloading
from steam over the internet.

Using an SSD for the cache is highly recommended if possible.
