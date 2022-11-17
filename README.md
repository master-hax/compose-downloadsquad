# compose-downloadsquad

a multi-container docker application to run an automated download pod, aptly named downloadsquad

## how to set it up
1. clone this repo
1. put your wg0.conf file into ./wireguard (or use a different tunnel entirely)
1. run docker-compose up

## how to use it
the web UIs of each container is accessible at port 80, depending on what hostname is passed in the HTTP request.

note: rtorrent & prowlarr are visble to containers outside the VPN as "external-proxy"
