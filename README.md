# Docker rtorrent-wireguard

Based on horjulf/rutorrent-autodl. Adds support for wireguard VPN connections. 

## Changes Made
Most of the changes from horjulf/rutorrent-autodl were made to ensure a wireguard connection is used by rtorrent.

### Summary of Changes
* `--cap-add NET_ADMIN` and `--sysctl net.ipv4.conf.all.src_valid_mark=1` are needed to use wireguard
* rtorrent will only start when wg0 is brought up successfully
* rtorrent will bind to wg0 (view rtorrent.rc to see how it dynamically gets the IP of wg0)
* nginx now uses htpasswd for authentication
* `/config/wireguard/wg0.conf` and `/config/nginx/.htpasswd` must be manually created (see respective usage sections)

## Usage

### docker create
```sh
docker create --name=rtorrent-wireguard \
-v <path for config>:/config \
-v <path for downloads>/downloads \
-e PGID=<GID> -e PUID=<UID> \
-e TZ=<TZ timezone> \
-p 80:80 -p 5000:5000 \
-p 51413:51413 -p 6881:6881/udp \
--cap-add NET_ADMIN \
--sysctl net.ipv4.conf.all.src_valid_mark=1 \
joepol/rtorrent-wireguard
```

After you create/run the container, you must create `/config/wireguard/wg0.conf` and `/config/nginx/.htpasswd` (see below)

### htpasswd
You will need to generate a .htpasswd file for authentication. It is strongly recommended to use `htpasswd` to generate this file within the container. 

For example:
`docker exec -it rtorrent-wireguard htpasswd -cb /config/nginx/.htpasswd abc yourpasswordhere`

### Wireguard
You will need to write a wireguard configuration file compatible with wg-quick at `/config/wireguard/wg0.conf`. More information and examples can be found here: https://wiki.archlinux.org/index.php/WireGuard#Client_config.

If you are looking for a VPN provider, Mullvad supports port forwarding and a handy wireguard config file generator. For more information on Mullvad or other VPN providers, check out https://www.privacytools.io/providers/vpn/#mullvad

## Parameters

`The parameters are split into two halves, separated by a colon, the left hand side representing the host and the right the container side.
For example with a port -p external:internal - what this shows is the port mapping from internal to external of the container.
So -p 8080:80 would expose port 80 from inside the container to be accessible from the host's IP on port 8080
http://192.168.x.x:8080 would show you what's running INSIDE the container on port 80.`

* `-p 80` - the port(s)
* `-p 5000` - the port(s)
* `-p 51413` - the port(s)
* `-p 6881/udp` - the port(s)
* `-v /config` - where config files are stored (for nginx, rtorrent, wireguard, etc.)
* `-v /downloads` - path to your downloads folder
* `-e PGID` for GroupID - see below for explanation
* `-e PUID` for UserID - see below for explanation
* `-e TZ` for timezone information, eg Europe/London
* `--cap-add NET_ADMIN` to add the NET_ADMIN capability which is necessary for bringing up wg0
* `--sysctl net.ipv4.conf.all.src_valid_mark=1` so we don't need to rely on wg-quick to set this value (since it won't be able to within the container)

It is based on alpine linux with s6 overlay, for shell access whilst the container is running do `docker exec -it rtorrent-wireguard /bin/bash`.

### User / Group Identifiers

Sometimes when using data volumes (`-v` flags) permissions issues can arise between the host OS and the container. We avoid this issue by allowing you to specify the user `PUID` and group `PGID`. Ensure the data volume directory on the host is owned by the same user you specify and it will "just work" ™.

In this instance `PUID=1001` and `PGID=1001`. To find yours use `id user` as below:

```sh
  $ id <dockeruser>
    uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)
```

## Setting up the application

Webui can be found at `<your-ip>:80` , configuration files for rtorrent are in /config/rtorrent, php in config/php and for the webui in /config/rutorrent/settings.

`Settings, changed by the user through the "Settings" panel in ruTorrent, are valid until rtorrent restart. After which all settings will be set according to the rtorrent config file (/config/rtorrent/rtorrent.rc),this is a limitation of the actual apps themselves.`

** Important note for unraid users or those running services such as a webserver on port 80, change port 80 assignment **

`** It should also be noted that this container when run will create subfolders ,completed, incoming and watched in the /downloads volume.**`

** The Port Assignments and configuration folder structure has been changed from the previous ubuntu based versions of this container and we recommend a clean install **

Umask can be set in the /config/rtorrent/rtorrent.rc file by changing value in `system.umask.set`

If you are seeing this error `Caught internal_error: 'DhtRouter::get_tracker did not actually insert tracker.'.` , a possible fix is to disable dht in `/config/rtorrent/rtorrent.rc` by changing the following values.

```shell
dht = disable
peer_exchange = no
```

## Info

* Shell access whilst the container is running: `docker exec -it rutorrent /bin/bash`
* To monitor the logs of the container in realtime: `docker logs -f rutorrent`

* container version number

`docker inspect -f '{{ index .Config.Labels "build_version" }}' rutorrent`

* image version number

`docker inspect -f '{{ index .Config.Labels "build_version" }}' horjulf/rutorrent-autodl`
