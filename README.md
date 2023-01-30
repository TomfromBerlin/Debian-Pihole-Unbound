# Debian-Pihole-Unbound-Fritzbox 7490
This repository is an attempt to show how Pihole and Unbound can be made to work on Debian (Bullseye).

## Why another tutorial about how to setup Pihole and Unbound

Well, despite the fact that there are many tutorials out there, none of them led me from a freshly installed operating system to a working lokal domain name server. I had to crawl the web, collecting information and hints. Then I had to try if they work for me but some of them were outdated, others simply wrong. But finally I made it and now I have a lokal domain name server and Pi-hole guarding my lokal network with all devices that are connected to that network.

One thing I learned the hard way: once there is a running system, make a backup. You installed und updated the OS? Make a backup. You installed Pi-hole and Unbound and it works? Make a backup. And after all the raspberry pi has been successfully integrated into your network? Make a backup and screenshots of your router configuration. Now I can just roll back and I will have my domain name server without the hassle of trial and error until it works properly.

## System Requirements

You need to have a lokal area network, a router as gateway to the internet, a Raspberry Pi and a second personal computer. The second pc is probably your workstation. You will also need either an external hard drive or an SD card. If you want to use an SD card, make sure that you can write to it from your workstation. I recommend using an external hard drive since SD cards are designed more for storing data than for operating as a system drive in a computer. When operating with an SD card, failures are therefore more likely.

The operating system on your workstation can be either Linux, Windows or iOS. It does not matter. Your workstation and in any case the Raspberry Pi have to be connected to your network via cables. You should not run the Raspberry Pi over WiFi as the connection may be unstable.

You will also need an image of the lastest Raspberry Pi OS or at least the rpi-imager.