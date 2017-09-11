# jemcgrath1/vpnrouter

[![](https://images.microbadger.com/badges/image/jemcgrath1/vpnrouter.svg)](https://microbadger.com/images/jemcgrath1/vpnrouter "Get your own image badge on microbadger.com")[![](https://images.microbadger.com/badges/version/jemcgrath1/vpnrouter.svg)](https://microbadger.com/images/jemcgrath1/vpnrouter "Get your own version badge on microbadger.com")[![](https://images.microbadger.com/badges/commit/jemcgrath1/vpnrouter.svg)](https://microbadger.com/images/jemcgrath1/vpnrouter "Get your own commit badge on microbadger.com")

This image creates an OpenVPN Client container that can be used to connect to your VPN provider (expressvpn / PIA etc). I built this to allow my other docker containers to treat this container like a vpnrouter.

This works really well, for providng the same VPN service to multiple containers and/or running multiple vpnrouter containers on the same host.

It is based on Alpine 3.6, for shell access whilst the container is running use `docker exec -it vpnrouter /bin/bash`.

![OpenVPN](https://raw.githubusercontent.com/jemcgrath1/vpnrouter/master/openvpntech_logo1.png)

## Usage

```
docker create \
--name=vpnrouter \
--net=vpn_nw \
--restart=always
--device /dev/net/tun
-v <path to vpn provider ovpn/config file(s)>:/vpn \
-v /etc/localtime:/etc/localtime:ro \
-e TZ=<timezone> \
-e OPENVPN_CONFIG_FILE=<name of ovpn file to use xyz.ovpn> \
-e OPENVPN_USERNAME=<username from vpn provider> \
-e OPENVPN_PASSWORD=<password from vpn provider> \
jemcgrath1/vpnrouter
```
  <br />

## Parameters

The following provides more details on the parameters of the containerL
* `--name` - Name the container
* `--net=vpn_nw` IMPORTANT - See Important Notes, you will need to create a *user-defined* network
* `--restart=always` - Apply a restart policy
* `--device /dev/net/tun` - Use the local tun device
* `-v /vpn` - The path where vpnrouter can find ovpn/config file(s) from your vpn provider
* `-v /etc/localtime:/etc/localtime:ro` - Read the localtime (Read Only)
* `-e TZ` - For timezone information eg Europe/London, etc
* `-e OPENVPN_CONFIG_FILE` - The full name of the ovpn or config file from vpn provider i.e. `xyz.ovpn`
* `-e OPENVPN_USERNAME` - The username from your vpn provider to use the ovpn/config file
* `-e OPENVPN_PASSWORD` - The password from your vpn provider to use the ovpn/config file



## Important Notes

* This container has been built to use a *user defined* network called vpn_nw. This needs to be created **prior** to starting the container.
This can be created by executing the docker command:  
 `docker network create --driver bridge vpn_nw`

* If you have connectivity issues, please try creating the container with an addtional dns flag as follows eg. Googles DNS `--dns 8.8.4.4`

* Running it on a NAS. This particular container works perfectly on my QNAP NAS without interfering with the existing VPN running on the NAS itself.

* Automatic login to VPN - you may need to edit your ovpn/config file(s) to add the following command to allow automatic login to your vpn provider

 * existing ovpn/config file contains the following line:  
`auth-user-pass`

 * Modify the auth-user-pass line in the ovpn/config file to include the following path:  
 `auth-user-pass  /config/openvpn-credentials.txt`  
 This tells the OpenVPN client to read the username/password you have provided as an environment vairiable, rather than waiting for user input from the keyboard.


**Connecting other containers to vpnrouter container**  

  :bangbang: To connect other containers to the vpn router, you will need to ensure that the other containers are started with `--net=container:vpnrouter` or the name of the vpnrouter container. Personally I like to name them by VPN exit location so I know where each router is directed.

  :bangbang: In addition, if you were planning to connect other containers (i.e transmission, sonarr etc) to the vpnrouter container, you **MUST** expose the ports required for application on the **vpnrouter** container before joing the containers.

   *i.e.*  To do this you would need to add the following additional ports to the vpnrouter create statement so that when you connect the transmission container vpnrouter makes the following ports are visible. For example you would add the following ports for Transmission:  
  `-p 9091:9091 -p 51413:51413 -p 51413:51413/udp `
 <br />
 <br />
 With these additional ports added to the vpnrouter container, you could then do something like (using the wonderful (linuxserver images):<br /><br />
 ` docker create --name=transmission_vpn --net=container:vpnrouter --restart=always -v <path to data>:/config -v <path to downloads>:/downloads -v <path to watch>:/watch -e PGID=<gid> -e PUID=<uid> -e TZ=<timezone> linuxserver/transmission:latest`
 
 The transmission web interface would then be available on TCP 9091 of the **vpnrouter** container, not the transmission container, and all the transmission traffic outside of your network would be going over the vpn!
 <br /><br />
 **Enjoy!!**


## Info

* To monitor the logs of the container in realtime

`docker logs -f vpnrouter`.

* Container version number

`docker inspect -f '{{ index .Config.Labels "build_version" }}' vpnrouter`

* Image version number

`docker inspect -f '{{ index .Config.Labels "build_version" }}' jemcgrath1/vpnrouter`

## Versions

+ **2017.09.4** - Initial Release - Tested with Expressvpn and PIA


