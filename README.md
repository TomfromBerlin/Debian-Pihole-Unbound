# Debian (bullseye) with Pi-hole & Unbound & Fritzbox 7490
This repository is an attempt to show how Pi-hole and Unbound can be made to work on Debian (Bullseye). You should grab a cup of coffee or tea as there is a lot to read.

## Why another tutorial about how to setup Pihole and Unbound

Well, despite the fact that there are many tutorials out there, none of them led me from a freshly installed operating system to a working lokal domain name server. I had to crawl the web, collecting information and hints. Then I had to try if they work for me but some of them were outdated, others simply wrong. But finally I made it and now I have a lokal domain name server and Pi-hole guarding my lokal network with all devices that are connected to that network.

One thing I learned the hard way: once there is a running system, make a backup. You installed und updated the OS? Make a backup. You installed Pi-hole and Unbound and it works? Make a backup. And after all the raspberry pi has been successfully integrated into your network? Make a backup and screenshots of your router configuration. Now I can just roll back and I will have my domain name server without the hassle of trial and error until it works properly.

## System Requirements

You need to have a lokal area network, a router as gateway to the internet, a Raspberry Pi and a second personal computer. The second pc is probably your workstation. You will also need either an external hard drive or an SD card. If you want to use an SD card, make sure that you can write to it from your workstation. I recommend using an external hard drive since SD cards are designed more for storing data than for operating as a system drive in a computer. When operating with an SD card, failures are therefore more likely.

The operating system on your workstation can be either Linux, Windows or iOS. It does not matter. Your workstation and in any case the Raspberry Pi have to be connected to your network via cables. You should not run the Raspberry Pi over WiFi as the connection may be unstable.

You will also need an image of the lastest Raspberry Pi OS or at least the rpi-imager. I recommend to download the image and using balenaEtcher to flash the drive.

First time you start you Pi with the new OS it will take some time. Later the boot process will be faster. Answer the questions about specific settings, e.g. ssh should be activated and possibly also VNC. At this point, a connected monitor will do just fine, as will a keyboard. Later, access will be via ssh, but that's not available to us yet.

By the way, if the installation was done with the rpi-imager, you will be asked in advance whether ssh should be activated. You will also be asked for the country and keyboard settings. In my case, it was still necessary to set these settings after the first boot.

At this point it might be a good idea to update the system and install Midnight Commander:
```
$apt update
$apt upgrade
$apt install mc
```
We will use Midnight Commander later for navigating through the file system. Almost all operations require elevated privileges, in other words, you must be root.

### Known Issues
With a freshly installed system with 'bullseye' you might run into the issue that hciuart.service failed to start. In this case `$systemctl status hciuart.service` (with elevated rights) shows this:

```
● hciuart.service - Configure Bluetooth Modems connected by UART
     Loaded: loaded (/lib/systemd/system/hciuart.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Sat 2023-02-11 12:08:05 CET; 41min ago
    Process: 422 ExecStart=/usr/bin/btuart (code=exited, status=1/FAILURE)
        CPU: 191ms

Feb 11 12:07:18 raspberrypi systemd[1]: Starting Configure Bluetooth Modems connected by UART...
Feb 11 12:08:06 raspberrypi btuart[481]: Initialization timed out.
Feb 11 12:08:06 raspberrypi btuart[481]: bcm43xx_init
Feb 11 12:08:06 raspberrypi btuart[481]: Flash firmware /lib/firmware/brcm/BCM4345C0.hcd
Feb 11 12:08:05 raspberrypi systemd[1]: hciuart.service: Control process exited, code=exited, status=1/FAILURE
Feb 11 12:08:05 raspberrypi systemd[1]: hciuart.service: Failed with result 'exit-code'.
Feb 11 12:08:05 raspberrypi systemd[1]: Failed to start Configure Bluetooth Modems connected by UART.
```

You can try to fix this with the following steps (as root):

1) `nano /etc/systemd/system/bluetooth.target.wants/bluetooth.service`
2) Change: the line `ExecStart=/usr/lib/bluetooth/bluetoothd` to `ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap`
3) save the file with `Ctrl-o`/`Strg-o` and exit nano wit `Ctrl-x`/`Strg-x`
3) Reload the systemd: `$systemctl daemon-reload`
4) Restart the bluetooth: `$service bluetooth restart`
5) Get the bluetooth status: `$service bluetooth status`

```
    bluetooth.service - Bluetooth service
       Loaded: loaded (/lib/systemd/system/bluetooth.service; enabled)
       Active: active (running) since Sat 2016-04-30 10:38:46 UTC; 6s ago
         Docs: man:bluetoothd(8)
     Main PID: 12775 (bluetoothd)
       Status: "Running"
       CGroup: /system.slice/bluetooth.service
               └─12775 /usr/lib/bluetooth/bluetoothd --noplugin=sap
```
If you don't want to overwrite the system bluetooth.service file, it's a good place to use a *.service.d override (you need elevated privileges again):

1) `$mkdir /etc/systemd/system/bluetooth.service.d/`
2) place in a new file with: `$touch /etc/systemd/system/bluetooth.service.d/01-disable-sap-plugin.conf`
3) edit with: `$nano /etc/systemd/system/bluetooth.service.d/01-disable-sap-plugin.conf`
4) save with `Ctrl-o`/`Strg-o`; exit nano with `Ctrl-x`/`Strg-x`.

Add the following three lines to the file:
```
[Service]
ExecStart=
ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap
```

_The simplest explanation is that it clears out the value of ExecStart= so we can override it rather than append to it. Some SystemD settings behave as an appended list when specifying them multiple times. The third line defines the new value of ExecStart=_

Then enter the following commands in the command line:

```
$sudo systemctl daemon-reload
$sudo systemctl restart bluetooth.service
```

(source: <https://raspberrypi.stackexchange.com/questions/40839/sap-error-on-bluetooth-service-status>)

This may solve the problem, in my case it doesn't. At least one error message is gone.

|What actually helped was a new entry in the /boot/config.txt file: `dtparam=krnbt`, wich gives the kernel the task of initializing the BT modem. There are already entries starting with dtparam=, I added the new one there and it seems to help. This will probably become the standard in the future from what I've heard.|
|:-|

## Installing Pihole and Unbound

Now that we have a freshly set up system and have fixed or ignored the above errors, we can now move on to the actual task: installing Pihole and Unbound.

### Pi-hole

At first we will install Pihole. This shouldn't be a problem. You can use the following command and you'll be fine: `curl -sSL https://install.pi-hole.net | bash`. Piping to bash is a controversial topic, as it prevents you from reading code that is about to run on your system. If you would prefer to review the code before installation, the developers provide alternative installation methods. You can find [here](https://docs.pi-hole.net/main/basic-install/) more information.

The developers suggest to configure the router after installing Pi-hole and they provide information about different models. Regarding the Fritzbox there is a walkthrough in [English](https://docs.pi-hole.net/routers/fritzbox/) and [German](https://docs.pi-hole.net/routers/fritzbox-de/). For other routers there are instructions too. I recommend to keep track of the changes, since Unbound need a few more steps and it may be necessary to bypass Pi-hole because internet access is temporarily blocked and it may usefull to have an internet connection during the configuration process.

First, let's feed Pi-hole to see if it works. Pi-hole requires lists of malicious web addresses. Such lists can be found [here](https://github.com/RPiList/specials) (mainly for German users, so the instructions are also in German), among other places.

To check if Pi-hole is working, you have to [configure](https://docs.pi-hole.net/main/post-install/) your router and Pi-hole accordingly. It is recommended to have a static IP for your Raspberry Pi, otherwise you will run into problems, at the latest when running Unbound as local DNS.

Also, I did NOT set up Pi-hole as a DHCP server. I leave this to the Fritzbox. The latter will get the data from Unbound later anyway.

Pi-holes DNS settings should look like this for now:

![pihole-settings-dns](https://user-images.githubusercontent.com/123265893/218263468-af768d92-d14d-4c1d-970b-97ae23dad615.png)

Of course you can also select any other available DNS. I prefer DNS.Watch, but that decision is up to you.

Now, Pi-hole should be operational and you can grab another cup of coffee or tea as we're only halfway through.

### Unbound

In order to get Unbound to cooperate, a few more steps are necessary than various instructions on the internet promise us. Just installing it is not enough, but it is obviously necessary. In our case we just use the version that can be found in the repositories. This is currently version 1.9.xx. On GitHub you'll find a more recent version ([1.17.1](https://github.com/NLnetLabs/unbound)), but using the package from the standard repo avoids compiling and updating issues.

|While we're at it: I can't say whether the steps shown here are necessary for later versions or whether a reconfiguration is even mandatory. Rumor has it that newer versions of Unbound no longer need certain changes described here. They may then be overwritten and everything runs like clockwork - or not. Time will tell.|
|:-|

