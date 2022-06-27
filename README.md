# transmission-tailscale
# transmission-tailscale [![Drone Build/Push to dockerhub](https://drone.beardedtek.com/api/badges/BeardedTek/docker-transmission-tailscale/status.svg)](https://drone.beardedtek.com/BeardedTek/docker-transmission-tailscale)

Extends linuxserver/tranmission with tailscale functionality to mount remote NFS via tailscale network. As of right now, it only works on amd64.

## FEATURES:
- Use an Exit Node
  - An exit node on tailscale effective lets you "place shift".  Your IP address will be that of the exit node to all outside services.

- Auto Pause Transmission on NFS disconnect
  - Transmission will pause all downloads when NFS disconnects
    - For this to function properly, create a file named MOUNTED in the base NFS directory.

- Auto Pause Transmission on Tailscale disconnect
  - Transmission will pause all downloads when tailscale goes down
    - Since we are mounting our NFS via tailscale, pause all downloads if tailscale goes down  This way it's not filling up the remote drive
  
## Weekly Builds
I have this set to build weekly as its based on linuxserver/transmission.  I want it to keep up to date with upstream.

## docker-compose:
```
version: "2.4"
services:
  transmission-tailscale:
    image: beardedtek/transmission-tailscale:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./tailscale:/var/lib/tailscale
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
      DOCKER_NETWORK_SUBNET: ${DOCKER_NETWORK_SUBNET:-192.168.251.0/24}
      TAILSCALE_AUTHKEY: ${TAILSCALE_AUTHKEY}
      TAILSCALE_HOSTNAME: sidecar-nefyoung
      TAILSCALE_TAGS: ${TAILSCALE_TAGS}
      TAILSCALE_ADVERTISE_ROUTE: ${TAILSCALE_ADVERTISE_ROUTE}
      TAILSCALE_ENABLE: ${TAILSCALE_ENABLE:-false}
      MOUNT_NFS: ${MOUNT_NFS:-false} #set to true to mount an nfs volume
      NAS_IP: ${NAS_IP} #ip of nfs server
      NAS_SHARE: ${NAS_SHARE} #share path
    command: "/init"


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

## TODO:
- exit node functionality
- arm64
- armhf