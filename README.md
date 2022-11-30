# compose-downloadsquad

a multi-container docker application to run an automated download pod, aptly named downloadsquad.

contains the following:
1. wireguard ([docker-wireguard](https://github.com/linuxserver/docker-wireguard)) but any VPN tunnel can be used like [qdm12/gluetun](https://github.com/qdm12/gluetun) or [dperson/openvpn-client](https://github.com/dperson/openvpn-client)
1. rtorrent behind the VPN container ([crazy-max/docker-rtorrent-rutorrent](https://github.com/crazy-max/docker-rtorrent-rutorrent))
1. prowlarr behind the VPN container ([linuxserver/docker-prowlarr](https://github.com/linuxserver/docker-prowlarr))
1. radarr ([linuxserver/docker-radarr](https://github.com/linuxserver/docker-radarr))
1. lidarr (https://github.com/linuxserver/docker-lidarr))/github.com/linuxserver/docker-prowlarr))
1. sonarr ([linuxserver/docker-sonarr](https://github.com/linuxserver/docker-sonarr))
1. readarr ([linuxserver/docker-readarr](https://github.com/linuxserver/docker-readarr))

features:
1. unix domain sockets for secure & efficient communicate around the VPN, without having to use the hacky docker links approach
1. all containers accessible from a single port

## how to set it up
1. clone this repo
1. configure the VPN tunnel (wg0.conf into ./wireguard or use a different tunnel entirely)
1. run docker-compose up

## how to use it
the web UIs of all containers is accessible at port 80, depending on what hostname is passed in the HTTP request.
add the following line to your hosts file to access the web UI easily (using the IP address of your docker host):
```
127.0.0.1 rtorrent.downloadsquad sonarr.downloadsquad lidarr.downloadsquad readarr.downloadsquad radarr.downloadsquad prowlarr.downloadsquad 
```
then you can visit the following in your browser:
* [http://rtorrent.downloadsquad](http://rtorrent.downloadsquad)
* [http://prowlarr.downloadsquad](http://rtorrent.downloadsquad)
* [http://radarr.downloadsquad](http://rtorrent.downloadsquad)
* [http://lidarr.downloadsquad](http://rtorrent.downloadsquad)
* [http://sonarr.downloadsquad](http://rtorrent.downloadsquad)
* [http://readarr.downloadsquad](http://rtorrent.downloadsquad)

give it 30 seconds or so for the nginx proxies to start up & make sure you are using http (no s)

note:
* rtorrent & prowlarr are visble to containers outside the VPN as "aggregator-proxy"
* containers outside the VPN are visible to rtorrent & prowlarr as "localhost"

note:
* you will likely want to set up port forwarding on the VPN tunnel container for rtorrent

note:
* this project runs out of the box but make sure you change the volumes to persistent ones if you want to use it long term

note:
* make sure to test that prowlarr & rtorrent are behind the VPN by running
```
docker exec -it compose-downloadsquad-rtorrent-1 curl api.ipify.org
```
&
```
docker exec -it compose-downloadsquad-prowlarr-1 curl api.ipify.org
```
* and optionally test to make sure e.g. sonarr is not behind the VPN
```
docker exec -it compose-downloadsquad-sonarr-1 curl api.ipify.org
```

optional:
* consider locking the docker image versions to specific versions rather than "latest" for supply chain security/consistency.

## how it works
we only want to pass `rtorrent` & `prowlarr` through the VPN, but they still need to communicate with containers outside the VPN. So we use [unix domain sockets](https://en.wikipedia.org/wiki/Unix_domain_socket) on a temporary docker volume for secure & efficient communication around the VPN + 2 instances of nginx to proxy requests through them.
