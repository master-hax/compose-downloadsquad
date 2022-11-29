# compose-downloadsquad

a multi-container docker application to run an automated download pod, aptly named downloadsquad.

features:
1. unix domain sockets for secure & efficient communicate around the VPN, without having to use the hacky docker links approach
1. all containers accessible from a single port

## how to set it up
1. clone this repo
1. put your wg0.conf file into ./wireguard (or use a different tunnel entirely. works with [qdm12/gluetun](https://github.com/qdm12/gluetun), [dperson/openvpn-client](https://github.com/dperson/openvpn-client), etc.)
1. run docker-compose up

## how to use it
the web UIs of all containers is accessible at port 80, depending on what hostname is passed in the HTTP request.
add the following to your hosts file to access the web UI easily (using the IP address of your docker host):
```
127.0.0.1 rtorrent.downloadsquad sonarr.downloadsquad lidarr.downloadsquad readarr.downloadsquad radarr.downloadsquad prowlarr.downloadsquad 
```

note:
* rtorrent & prowlarr are visble to containers outside the VPN as "aggregator-proxy"
* containers outside the VPN are visible to rtorrent & prowlarr as "localhost"

## how it works
we only want to pass `rtorrent` & `prowlarr` through the VPN, but they still need to communicate with containers outside the VPN. So we use [unix domain sockets](https://en.wikipedia.org/wiki/Unix_domain_socket) on a temporary docker volume for secure & efficient communication around the VPN + 2 instances of nginx to proxy requests through them.
