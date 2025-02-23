#testbed-server

## Background
This repository is meant as documentation for my home servers, with information on hardware, software, and services I well be self-hosting, in an attempt to learn more about systems administration, networking, security, and specific software like Docker and Proxmox. This is a shared network with my partner and roommates and there are limitations with both hardware and the level of tinkering. My goal is to utilize primarily open source software and cheaper hardware I can get second hand off FaceBook Marketplace or eBay to tinker and cut back on my reliance of third party services such as cloud providers, streaming services, and other subscription services. I will also be exploring penetration testing and attempting to uncover security vulnerabilities in my home servers to learn how to defend myself and systems from such risks. Most of these ideas were sprouted by poor Wi-Fi connection to my bedroom and initially using a Raspberry Pi 3B+ as an access point.


ansible and scripts to setup servers

wireshark
openstack
kubernetes
tcpdump
scapy
iperf

## Network Diagram
This is a network diagram showing the hardware and planned services that will be hosted (subject to change). VLAN with roommates, Router, APs, 
![alt text](example-network.png)

physical connection
vlans and layer 2 domains
subnets and layer 3 connectivity

## Router
Our current place is on Bell's Fibre network and uses a Bell Gigahub acting as our modem and router. I want to use another machine as our router running OPNSense. 
My initial plan was to put the Gigahub into [bridge mode](https://pon.wiki/guides/bridge-the-bce-inc-giga-hub/), however it seems that a recent firmware update has disabled this feature for non-business customers. 
I decided to go with Advanced DMZ since it is simple to setup, and forward the network traffic from the Gigahub to the OPNSense box. From my understanding, this should minimize most Double NAT issues and set OPNSense to control routing and firewall duties.

### Bell model
Access the GUI at https://192.168.2.1
Enable Advanced DMZ and select the OPNSense router's mac address
Disable Wi-Fi

### OPNSense
The GUI can be found at https://192.168.1.1

#### Unbound DNS
Enhance security and privacy, resolves DNS queries.
Enabled by default in OPNSense. How to use with Bell Gigahub modem?

#### [AdGuard Home](https://github.com/AdguardTeam/AdGuardHomehttps://github.com/AdguardTeam/AdGuardHome)

https://0x2142.com/how-to-set-up-adguard-on-opnsense/

We are installing AdGuard Home (AGH) onto the OPNSense box to get network-wide blocking of ads and tracking. Focus on filtering and maintaining DNS cache.
- In the GUI, enable SSH by going to System -> Settings -> Administration -> Enable Secure Shell
- SSH into the router
`ssh root@192.168.1.1`
- Once signed in, select option 8) Shell
- Fetch [opn-repo](https://github.com/mimugmail/opn-repo), since AGH is not available by default in the OPNSense plugins repository
`fetch -o /usr/local/etc/pkg/repos/mimugmail.conf https://www.routerperformance.net/mimugmail.conf`
`pkg update`
- In the GUI, go to System -> Firmware -> Plugins and install "os-adguardhome-maxit"
- Then go to Services -> Adguardhome -> General and check the box to enable it

The AGH GUI can be found at http://192.168.1.1:3000
- In the Admin Web Interface section, set the Listen interface to the LAN network (in my case, igb0 - 192.168.1.1) and the port to 3000
- Do the same for the DNS server section, but set the port to 65353
- Setup a username and password
- Login to the AGH dashboard

Navigate to Filters -> DNS blocklists
- OISD Blocklist Small
- HaGeZi's Pro++

Back to OPNSense GUI
- Services -> ISC DHCPv4 -> DNS servers -> "192.168.1.1"
- Services -> Unbound DNS -> Query Forwarding 
Add new, set server IP and port to AGH (192.168.1.1 and 65353)

#### WireGuard vs OpenVPN vs IPsec
WireGuard is UDP only, but faster encryption
OpenVPN supports TCP and UDP, 
Every platform supports IPsec, not too useful for my use case

#### VLAN


## Managed Switch (Netgear M4100-26G-POE)
[Documentation from Netgear](https://www.netgear.com/support/product/m4100-26g-poe%20gsm7226lpv1h1/#docs)

### VLAN

## TrueNAS Scale
One pool with RAIDZ1
https://www.youtube.com/watch?v=3T5wBZOm4hY

[TrueNAS Scale: A Step-by-Step Guide to Dataset, Shares, and App Permissions](https://www.youtube.com/watch?v=59NGNZ0kO04)

### Syncthing
[Video by Lawrence Systems](https://www.youtube.com/watch?v=PCYvsLSStbA)

### NextCloud

### Paperless NGX

### 2FAuth

### HomePage

### Portainer

### Calibre

### Calibre-web

### FreshRSS

### Immich

### QBitTorrent

### SearXNG

### VaultWarden

## Proxmox Cluster

### Docker and Portainer in Proxmox
https://www.youtube.com/watch?v=wrlukx-QYRw&t=281s

### Proxmox setup
https://technotim.live/posts/first-11-things-proxmox/

### Home Assistant OS in Proxmox
https://www.derekseaman.com/2023/10/home-assistant-proxmox-ve-8-0-quick-start-guide-2.html

### Stacks = Docker compose in Portainer
https://www.portainer.io/blog/stacks-docker-compose-the-portainer-way

## Access Points (WS-AP3825i)

### Resources
- [Reddit: Extreme Networks WS-AP3825i setup tutorial](https://www.reddit.com/r/openwrt/comments/1e0otf7/extreme_networks_wsap3825i_setup_tutorial/)
- [OpenWrt: Extreme Networks WS-AP3825i / WS-AP3825E](https://openwrt.org/toh/extreme_networks/ws-ap3825i)
- [OpenWrt: Wi-Fi Extender/Repeater with Bridged AP over Ethernet](https://openwrt.org/docs/guide-user/network/wifi/wifiextenders/bridgedap)

### connect a serial usb to ethernet cable from host device to the access point's console port
- `sudo picocom -b 115200 /dev/ttyUSB0`

### create a tftp server on host device and download an openwrt image
- `cd $(mktemp -d)`
- `curl https://downloads.openwrt.org/releases/23.05.5/targets/mpc85xx/p1020/openwrt-23.05.5-mpc85xx-p1020-extreme-networks_ws-ap3825i-initramfs-kernel.bin -o ws-ap3825i-initramfs.bin`
- `cp ws-ap3825i-initramfs.bin initramfs.bin`
- `sudo dnsmasq -d --enable-tftp --port 0 --tftp-root $(pwd)`

### configure and load the firmware over tftp (OpenWrt >= version 23)
- `setenv ramboot_openwrt "setenv ipaddr <ipv4 client address>; setenv serverip <tftp server address>; tftpboot 0x2000000 initramfs.bin; interrupts off; bootm 0x2000000;"`
- `setenv boot_openwrt "bootm 0xEC000000;"`
- `setenv bootcmd "run boot_openwrt";`
- `saveenv`
- `run ramboot_openwrt`

### upgrade the system
- `cd /tmp`
- `wget <latest firmware OpenWrt upgrade link above>`
- `sysupgrade -n /tmp/openwrt-mpc85xx-p1020-extreme-networks_ws-ap3825i-squashfs-sysupgrade.bin`

Reboot the system

### edit the config files using the ones provided in ./openwrt/etc/config then reboot the system

### Access LuCI by connecting to the static IP address of the access point
1. go to Network -> Wireless, and edit the SSIDs
2. go to the Wireless Security tab under interface configuration
3. set encryption to WPA2-PSK and set the Key (password)
4. go to the WLAN roaming tab and enable 802.11r Fast Transition with FT over the Air

## Kali Linux VM
### nmap
Network scan of IP addresses to diagnose issues with the wireless access points after migrating to Advanced DMZ and the OPNSense router.

## TODO
- UPS
- Upgrade storage
- Upgrade Jellyfin server to 7th gen intel or newer for transcoding
- Ollama/local LLM server with GPU (Tesla P40/P4/P100/P102-100, VRAM)
