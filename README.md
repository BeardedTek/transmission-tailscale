# transmission-tailscale 
[![Release](https://img.shields.io/github/v/release/beardedtek/transmission-tailscale)](https://github.com/BeardedTek/transmission-tailscale/releases)
[![ReleaseDate](https://img.shields.io/github/release-date/beardedtek/transmission-tailscale)](https://github.com/BeardedTek/transmission-tailscale/releases)
[![Build Status](https://drone.beardedtek.com/api/badges/BeardedTek/transmission-tailscale/status.svg?ref=refs/heads/main)](https://drone.beardedtek.com/BeardedTek/transmission-tailscale)
[![Image Size](https://img.shields.io/docker/image-size/beardedtek/transmission-tailscale/latest)](https://hub.docker.com/r/beardedtek/transmission-tailscale)
[![Pulls](https://img.shields.io/docker/pulls/beardedtek/transmission-tailscale)](https://hub.docker.com/r/beardedtek/transmission-tailscale)
[![license](https://img.shields.io/github/license/beardedtek/transmission-tailscale)](https://github.com/BeardedTek/transmission-tailscale/blob/0.1.0/LICENSE)
[![twitter-follow](https://img.shields.io/twitter/follow/beardedtek?style=social)](https://twitter.com/intent/user?screen_name=beardedtek)

Extends linuxserver/tranmission with tailscale functionality to mount remote NFS via tailscale network. As of right now, it only works on amd64.

## Features:
- Use tailscale to remotely and securely view your instance of nefarious outside your network via [Tailscale](https://tailscale.com)
- Placeshift your presence using an [exit node](https://tailscale.com/kb/1103/exit-nodes/)
- Mount an NFS share inside docker with or without tailscale
- Use with programs such as [nefarious](https://github.com/lardbit/nefarious), [radarr](https://radarr.video), [sonarr](sonarr.tv) to add tailscale capabilities
  
## Weekly Builds
I have this set to build daily as its based on linuxserver/transmission.  I want it to keep up to date with upstream.

## docker-compose:
```
version: "2.4"
services:
  transmission-tailscale:
    image: beardedtek/transmission-tailscale:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./tailscale-data:/var/lib/tailscale
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - net_admin
      - sys_module
      - sys_admin
    sysctls:
      - net.ipv6.conf.all.disable_ipv6=${TAILSCALE_IPV6_DISABLED:-1}  # 0=ipv6 enabled, 1=ipv6 disabled
      - net.ipv4.ip_forward=1
      - net.ipv6.conf.all.forwarding=${TAILSCALE_IPV6_FORWARDING:-1}
    environment:
      TAILSCALE_ENABLE: ${TAILSCALE_ENABLE:-false}
      TAILSCALE_ADVERTISE_ROUTES: ${TAILSCALE_ADVERTISE_ROUTES}
      TAILSCALE_AUTHKEY: ${TAILSCALE_AUTHKEY}
      TAILSCALE_EXIT_NODE: ${TAILSCALE_EXIT_NODE}
      TAILSCALE_HOSTNAME: ${TAILSCALE_HOSTNAME}
      TAILSCALE_TAGS: ${TAILSCALE_TAGS}

      NFS_ENABLE: ${NFS_ENABLE:-false}
      NFS_IP: ${NFS_IP}
      NFS_SHARE: ${NFS_SHARE}
      NFS_MOUNT_POINT: ${NFS_MOUNT_POINT}


    networks:
      transmission-tailscale:
          ipv4_address: ${TRANSMISSION_IP:-192.168.251.201}

networks:
  transmission-tailscale:
    driver: bridge
    ipam:
      config:
        - subnet: ${SUBNET:-192.168.251.0/24}
          gateway: ${GATEWAY:-192.168.251.254}
```
#### .env file
```
###############################################################################
# Tailscale Settings
###############################################################################

# Enable / Disable Tailscale Functionality
TAILSCALE_ENABLE=true

# Override auto detection of docker network (ex: 192.168.1.0/24)
TAILSCALE_ADVERTISE_ROUTES=

# Use an Auth Key
# Obtain auth key at: https://login.tailscale.com/admin/settings/keys
TAILSCALE_AUTHKEY=

# "Placeshift" your presence.  You IP will appear as the public address of the
# node you enter here.  Supply the Tailscale IP address of the exit node you
# want to use
TAILSCALE_EXIT_NODE=

# Hostname as it will appear within your tailnet
TAILSCALE_HOSTNAME=transmission-tailscale

# Tag this tailscale instance.  You must define tags prior to use.
# See https://tailscale.com/kb/1068/acl-tags/#defining-a-tag
# Tags are in format tag:<tagname>
TAILSCALE_TAGS=

###############################################################################
# NFS Settings
###############################################################################

# Enable NFS inside container
NFS_ENABLE=true

# IP/fqdm of NFS Server
NFS_IP=192.168.1.111

# Full path to NFS Share
NFS_SHARE=/path/to/share

# Mount point inside container
NFS_MOUNT_POINT=/downloads
```

## TODO:
- exit node functionality
- arm64
- armhf