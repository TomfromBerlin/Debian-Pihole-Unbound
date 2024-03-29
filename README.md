| ![Views](https://img.shields.io/endpoint?color=green&label=Views&logo=Debian&logoColor=red&style=plastic&url=https%3A%2F%2Fhits.dwyl.com%2FTomfromBerlin%2FDebian-Pihole-Unbound) | ![Unique Viewers](https://img.shields.io/endpoint?color=green&label=Unique%20Viewers&logo=Raspberry%20Pi&logoColor=pink&style=plastic&url=https%3A%2F%2Fhits.dwyl.com%2FTomfromBerlin%2FDebian-Pihole-Unbound.svg%3Fshow%3Dunique) | ![DEBIAN](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white) | ![RaspberryPi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=for-the-badge&logo=Raspberry%20Pi&logoColor=white) | ![FRITZ!box 7490](https://user-images.githubusercontent.com/123265893/219904891-ab50d92a-ad69-480a-adc3-dd37b8a21534.svg) | [German Version](/../../../../TomfromBerlin/Debian-Pihole-Unbound/blob/main/README_de.md)|
 |-|-|-|-|-|-|
 
# _Debian (bullseye) with Pi-hole & Unbound & Fritzbox 7490_
This is an attempt to show how Pi-hole and Unbound can be made to work on Debian (Bullseye). You should grab a cup of coffee or tea as there is a lot to read.

| Disclaimer: The way described here worked for me. I can't say if it works for you too. But maybe this description will help you to find the right path for you. However, there is no warranty of any kind. Good luck with it. |
|:-|

A further description of how it all works can be found here: <https://docs.pi-hole.net/guides/dns/unbound/#>

## _Why another tutorial about how to setup Pihole and Unbound_

Well, despite the fact that there are many tutorials out there, none of them led me from a freshly installed operating system to a working lokal domain name server. I had to crawl the web, collecting information and hints. Then I had to try if they work for me but some of them were outdated, others simply wrong. But finally I made it and now I have a lokal domain name server and Pi-hole guarding my local network with all devices that are connected to that network.

One thing I learned the hard way: once there is a running system, make a backup. You installed und updated the OS? Make a backup. You installed Pi-hole and Unbound and it works? Make a backup. And after all the raspberry pi has been successfully integrated into your network? Make a backup and screenshots of your router configuration. Now I can just roll back and I will have my domain name server without the hassle of trial and error until it works properly.

## _System Requirements_

You need to have a lokal area network, a router as gateway to the internet, a Raspberry Pi and a second personal computer. The second pc is probably your workstation. You will also need either an external hard drive or an SD card. If you want to use an SD card, make sure that you can write to it from your workstation. I recommend using an external hard drive since SD cards are designed more for storing data than for operating as a system drive in a computer. When operating with an SD card, failures are therefore more likely.

The operating system on your workstation can be either Linux, Windows or iOS. It does not matter. Your workstation and in any case the Raspberry Pi have to be connected to your network via cables. You should not run the Raspberry Pi over WiFi as the connection may be unstable.

You will also need an image of the lastest Raspberry Pi OS or at least the rpi-imager. I recommend to download the image and using balenaEtcher to flash the drive.

First time you start your Pi with the new OS it will take some time. Later the boot process will be faster. Answer the questions about specific settings, e.g. ssh should be activated and possibly also VNC. At this point, a connected monitor will do just fine, as will a keyboard. Later, access will be via ssh, but that's not available to us yet.

By the way, if the installation will be done with the rpi-imager, you will be asked in advance whether ssh should be activated. You will also be asked for the country and keyboard settings. In my case, it was still necessary to set these settings after the first boot.

From this point on, everything can be done via ssh.

Now it might be a good idea to update the system and, perhaps, install Midnight Commander:

```
$apt update
$apt upgrade
$apt install mc
```

You can use Midnight Commander later for navigating through the file system. In my opinion this gives a better overview and the navigation is a bit faster (e.g. `Ctrl-Page up` / `Strg-Bild hoch` jumps to the next higher directory - with the root directory being the highest. Additionally, with the hotkey F3 you have a file viewer, and with F4 an editor at hand. Almost all operations require elevated privileges, in other words, you must be root.

You can start Midnight Commander with `sudo mc` to prevent typing sudo every time you want to edit a file. But be careful. In this mode you can cause a lot of mischief by accident. The shortcut `Ctrl-o` puts Midnight Commander in the background, pushing it again brings it back in the foreground. This is useful if you need to use the console and want to read the output of a command, e.g., `$systemctl status hciuart.service` wich we want to invoke now.

### _Known Issues_

|❗| Update May, 17th 2023 | It is possible that additionally connected Bluetooth adapters (mouse, keyboard, audio) may conflict with the built-in adapter. In case of problems, it might help to plug the additional adapter into the USB port only after booting. | OS: RaspiOS - Distribution: Debian GNU/Linux 11 (bullseye) 11 - Kernel Release: 6.1.21-v8+ - Kernel Version: #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 - Architecture: aarch64 |❗|
|-|-|:-|:-|-|

|❗| Update April, 30th 2023 | After the system update I had the same problems with bluetooth as described below. So I tried to reload bluetooth.service after the boot sequence finished, which failed. I decided to comment out the added entry in /etc/boot/config.txt `dtparam=krnbt`. Well, the system booted without any complaints regarding Bluetooth. | OS: RaspiOS - Distribution: Debian GNU/Linux 11 (bullseye) 11 - Kernel Release: 6.1.21-v8+ - Kernel Version: #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 - Architecture: aarch64 |❗|
|-|-|:-|:-|-|

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

The output seems to be okay now.

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

1) Creat a new directory with `$mkdir /etc/systemd/system/bluetooth.service.d/`
2) place in a new file with: `$touch /etc/systemd/system/bluetooth.service.d/01-disable-sap-plugin.conf`
3) edit with: `$nano /etc/systemd/system/bluetooth.service.d/01-disable-sap-plugin.conf`
     Add the following three lines to the file:
     ```
     [Service]
     ExecStart=
     ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap
     ```
     
4) save with `Ctrl-o`/`Strg-o`; exit nano with `Ctrl-x`/`Strg-x`.

_(Simple) explanation: it clears out the value of ExecStart= so the value can be overwritten instead of being appended to it. Some SystemD settings behave as an appended list when specifying them multiple times. The third line defines the new value of ExecStart=_

Then enter the following commands in the command line:

```
$systemctl daemon-reload
$systemctl restart bluetooth.service
```

(source: <https://raspberrypi.stackexchange.com/questions/40839/sap-error-on-bluetooth-service-status>)

Now, hciuart.service should be up and running, but well, that's a false flag. At least one error message is gone. However, the next start is coming and you will face the same problem: hciuart.service fails to start. The problem is more likely that the BT modem is initialized too late.

|‼️| What actually helps is a new entry in the _/boot/config.txt_ file: `dtparam=krnbt`, wich gives the kernel the task of initializing the BT modem. There are already entries starting with dtparam=, I added the new one there and it seems to help. This will probably become the standard in the future from what I've heard. At this point, a big thank you [@pelwell](/../../../../pelwell) for the [crucial hint](https://github.com/RPi-Distro/pi-bluetooth/issues/25#issuecomment-1426768853). |‼️|
|:-:|:-|:-:|

## _Installing Pihole and Unbound_

Now that we have a freshly set up system and fixed or ignored the errors mentioned above, we can now move on to the actual task: installing Pihole and Unbound. But before that we should

### _Set a static IP address for the Raspberry Pi_

This is done via the router interface. You can reach it by entering http://fritz.box or http://ip-address-of-the-router in the address bar of your preferred browser.
Then navigate to the following screen:

![fritzbox_network_connections](https://user-images.githubusercontent.com/123265893/218282027-874f4508-c003-4fc8-ab3f-f162ed180dd1.PNG)

Find your Raspberry Pi and click on the pencil icon on the right side. You will now see the settings regarding your Pi. Choose the IP address you want for your Pi and make it static. We will see that IP address later.

![fritzbox_raspberrypi_settings](https://user-images.githubusercontent.com/123265893/218282458-1723be36-b3dd-47dd-bd05-67a59734c598.PNG)

### _Pi-hole_

Now we will install Pi-hole. This shouldn't be a problem. You can use the following command and you'll be fine: `curl -sSL https://install.pi-hole.net | bash`. Piping to bash is a controversial topic, as it prevents you from reading code that is about to run on your system. If you would prefer to review the code before installation, the developers provide alternative installation methods. You can find [here](https://docs.pi-hole.net/main/basic-install/) more information.

The developers suggest to configure the router after installing Pi-hole and they provide information about different models. Regarding the Fritzbox there is a walkthrough in [English](https://docs.pi-hole.net/routers/fritzbox/) and [German](https://docs.pi-hole.net/routers/fritzbox-de/). For other routers there are instructions too. I recommend to keep track of the changes, because internet access may temporarily blocked and it may usefull to have an internet connection during the configuration process.

First, let's feed Pi-hole to see if it works. Pi-hole requires lists of malicious web addresses. Such lists can be found [here](/../../../../RPiList/specials/blob/master/Blocklisten.md) (mainly for German users, so the instructions are also in German), among other places.
Log in to your Pi-hole and click on 'ADLISTS'. You should now see this page:

![pi-hole_adlists](https://user-images.githubusercontent.com/123265893/218336144-76b6f54d-b967-422d-bfd8-02afa6872aeb.png)

You can add multiple lists by separating each entry with a space. In other words: you can mark all lists at once, rightclick and copy them to the clipboard. Then you can paste them in the address field in Pi-hole. After adding the lists you have to run `pihole -g` to update your gravity database. You can do this also within the Pi-hole interface, but then you have to keep this site open until the update is done. I recommend the command line.

If you forgot the password for Pi-hole that has been shown during the installation process just click `Forgot password`. You'll get instructions how to set a new password. An ssh connection would be very handy this moment.

By the way, a cron job is created during the installation process to update your Gravity database regularly. The command `ls -l /etc/cron*` shows all running cron jobs. The output looks like this:

![pihole_cron](https://user-images.githubusercontent.com/123265893/232343061-90c3ba6e-e742-4012-897b-3f7cdf2605a6.png)

To make sure Pi-hole is working, you have to [configure](https://docs.pi-hole.net/main/post-install/) your router and Pi-hole accordingly. It is recommended to have a static IP for your Raspberry Pi, otherwise you will run into problems, at the latest when running Unbound as local DNS. But we've already done that, so that shouldn't be an issue anymore.

Also, I did NOT set up Pi-hole as a DHCP server. I leave this to the Fritzbox. Assuming your Raspberry Pi isn't working and Pi-hole is DHCP server, then everyone on the network is blind and you probably need to reset your router. That would be really annoying.

Pi-holes DNS settings should look like this for now:

![pihole-settings-dns](https://user-images.githubusercontent.com/123265893/218263468-af768d92-d14d-4c1d-970b-97ae23dad615.png)

Of course you can also select any other available DNS. I prefer DNS.Watch, but that decision is up to you.

Now, Pi-hole should be operational and you can grab another cup of coffee or tea as we're only halfway through.

### _Unbound_

|❗| Update May, 17th 2023 | Newer distributions of RaspiOS include the Network Manager for network management in addition to dhcpd. This can be selected with the tool `raspi-config`. The changes to the files described below are then probably no longer necessary. I selected the network manager after making the changes and everything continues to work without having to make any further adjustments. There are differences with the port, which was set to `5335` for `dhcpd`. If the command `dig nasa.gov 127.0.0.1 -p 5335` is entered after the change, an error message occurs. The port `53` must be specified here, i.e. `dig nasa.gov 127.0.0.1 -p 53`. Unbound continues to be used as DNS and also answers corresponding queries via this port. But, as I said, I only had to specify this in the command. How it behaves on a freshly installed system, I cannot say at the moment. | OS: RaspiOS - Distribution: Debian GNU/Linux 11 (bullseye) 11 - Kernel Release: 6.1.21-v8+ - Kernel Version: #1642 SMP PREEMPT Mon Apr  3 17:24:16 BST 2023 - Architecture: aarch64 |❗|
|-|-|:-|:-|-|

In order to get Unbound to cooperate, under Debian Bullseye a few more steps are necessary than various instructions on the internet promise us. Just installing it is not enough, but it is obviously necessary. In our case we just use the version that can be found in the Debian default repositories. This is currently version 1.9.xx. On GitHub you'll find a [more recent version](/../../../../NLnetLabs/unbound), but using the package from the standard repo avoids compiling and updating issues.

|While we're at it: I can't say whether the steps shown here are necessary for later versions or whether a reconfiguration is even mandatory. Rumor has it that newer versions of Unbound no longer need certain changes described here. They may then be overwritten and everything runs like clockwork - or not. Time will tell.|
|:-|

Here's an overview of the files that will play a role in some way and the directories in which to find them:

```
+-etc (directory)
   |   dhcpcd.conf (file)
   |   resolv.conf (file)
   |   resolvconf.conf (file)
   |   
   +---dnsmasq.d (subdirectory)
   |       99-edns.conf (file)
   |       
   \---unbound (subdirectory)
       \---unbound.conf.d (subdirectory)
               pi-hole.conf (file)
               resolvconf_resolvers.conf (file)
               root-auto-trust-anchor-file.conf (file)
```

As you can see, the whole thing takes place in the /etc/ directory or in subdirectories of /etc/. As a result, elevated privileges are required for each write operation. I won't mention this every time, you should already know the drill.

In the following we go through each individual file step by step and in the end Unbound should work well with Pi-hole and answer the DNS queries accordingly.

#### /etc/dhcpcd.conf

To this file probably only one or two changes are necessary, since during the installation of Pi-hole most of the required entries are written. At the very end of this file you'll find the follwing line:

```
# fallback to static profile on eth0
```
Below this line should be the IP addresses of your Raspberry Pi, your router, and the DNS. It will look like this (of course, it's not really a table):

| Entry | Description |
|:-|:-|
| #interface eth0 | do not touch this line |
| #fallback static_eth0 | do not touch this line |
| interface eth0 | interface name, "eth" indicates a cable connection using an ethernet card, where the number at the end indicates the number of the interface; you shouldn't use a WiFi connection as it is considered unstable|
| static ip_address=192.168.xxx.xxx/24 | <--- this has to be the IP address of your Raspberry Pi, under which the little rascal can be reached in the home network |
| static routers=192.168.xxx.xxx | <--- this has to be the IP address of your router, default is 192.168.178.1 |
| static domain_name_servers=127.0.0.1#5335 | <--- this will be the IP address of the DNS (it has to be the localhost addess); note the port address behind the IP, this is important and should be included if not present; it probably won't be there because no one knows yet that this port is needed |

The static IP address and the address of your router depend on the home network address space specified in the router. These settings must match the settings in your router. The address space between 192.168.0.1 and 192.168.255.254 is usually used for home networks and not for public networks. In this way, every device and every application "knows" whether it is in the home network or not. The Fritzbox uses this as default setting, so it might look similar at your site and only the IP address for the Raspberry Pi has to be changed, if its not correct. The port address after 127.0.0.1 has to be inserted. Now, we're done with this file.

When you have made the changes, do not restart the dhcp server since this may lead to a (temporary) loss of the internet connection. We will do a reboot later, when everything is done.

#### _/etc/resolv.conf_

This file contains just two lines:

```
# Generated by resolvconf
nameserver 127.0.0.1
```

The first line tells us that this is a generated file, so content changes does not remain. But we see, the port address is not specified here. Therefore it must be included in dhcpcd.conf so that it is advertised in conjunction with the IP address 127.0.0.1. Unbound expects the requests via this port, otherwise the DNS will not work.

As a reminder: at the moment the Fritzbox is still our DNS and it asks DNS.Watch if it doesn't know the requested address itself. But we want to change that, so lets continue.

#### _/etc/resolvconf.conf_

Recent Debian-based OS releases auto-install a package called `openresolv`, which will cause unexpected behaviour for Pi-hole and Unbound. Openresolv's service/config instructs resolvconf to write unbound's own DNS service at nameserver 127.0.0.1, but without the 5335 port, into the file `/etc/resolv.conf` (see previous section). That `/etc/resolv.conf` file is used by local services/processes to determine DNS servers configured. You need to remove openresolv and disable the service, or modify the configuration file to work-around the misconfiguration.

There are two options to solve the problem and some will choose

##### _The hard way_

just in case `unbound-resolvconf.service` is needed someday. In this case you need to modify `/etc/resolvconf.conf`. Almost at the end of the file you will find the following line:

`unbound_conf=/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`

Just comment that line out like this:

`#unbound_conf=/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`

This step is necessary because resolvconf creates an unwanted file when the entry is active. After the entry has commented out the unwanted file should be renamed

`mv /etc/unbound/unbound.conf.d/resolvconf_resolvers.conf resolvconf_resolvers.conf.backup`

or deleted

`rm /etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`

By simply removing the file and restart Unbound then it works correctly as a recursive DNS. The problem is that after a while this file is being auto generated again, this didn't happen in Buster but it is happening now in Bullseye because of the `openresolv` package. To prevent this, the /etc/resolvconf.conf file must be modified as described above, or you choose

##### _The easy way_

Check if unbound-resolvconf.service is active with `$systemctl status unbound-resolvconf.service`

If it's running (it probably will on Bullseye), just disable it with the following commands:

```
$systemctl disable unbound-resolvconf.service
$systemctl stop unbound-resolvconf.service
```

Then delete the file

`/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf` with

`rm /etc/unbound/unbound.conf.d/resolvconf_resolvers.conf`.

More information can be found [here](https://docs.pi-hole.net/guides/dns/unbound/#disable-resolvconf-entry-for-unbound-required-for-debian-bullsye-releases).

#### _/etc/dnsmasq.d/99-edns.conf_

The developers suggest considering adding `edns-packet-max=1232` to a configuration file like `/etc/dnsmasq.d/99-edns.conf` to signal FTL to respect the limit.
```
$touch /etc/dnsmasq.d/99-edns.conf
$nano /etc/dnsmasq.d/99-edns.conf
```
Insert `edns-packet-max=1232`, save the file and exit the editor, or use Midnight Commander to perform these operations.

#### _/etc/unbound/unbound.conf.d/pi-hole.conf_
 
This file contains settings wich can be found [here](https://docs.pi-hole.net/guides/dns/unbound/#configure-unbound). Basically, you can copy/paste the content. The only(!) change I made is `do-ip6: no` to `do-ip6: yes`. For some reason Unbound (ver. 1.9.x) seems to want to use ip6. After a long search I found a hint in the depths of the internet that this could be the reason why Unbound is not playing. After changing that entry it worked. This may be due to the fact that this is also activated by default in the Fritzbox and I have not switched it off. But it gets weird later when it comes to telling Pi-hole to use Unbound as DNS.

#### _/etc/unbound/unbound.conf.d/resolvconf_resolvers.conf(.backup)_

This is the culprit. Delete or rename the file and disable the unbound-resolvconf.service, or at least manipulate /etc/resolvconf.conf (see [here](/../../../../TomfromBerlin/Debian-Pihole-Unbound/blob/main/README.md#the-hard-way)) to prevent its resurrection.

#### _/etc/unbound/unbound.conf.d/root-auto-trust-anchor-file.conf_

The root anchor key file, that is read in and written out. Default is /usr/share/dns/root.key. If the file does not exist, or is empty, a builtin root key is written to it. If you have installed Unbound from the Debian repository the package unbound-anchor has been installed as a dependency, so this file should already exist and you don't need to touch it.

Also, you don't need to download the root.hints file if you installed Unbound from the Debian repositories as it will be installed and updated automatically. Unbound uses this file to bootstrap domain resolution.

## _Pi-hole and Unbound_

Now we can tell Pi-hole, that we have a local DNS. From your workstation open Pi-hole in your webbrowser using the following address:

`http://IP-address-of-the-raspberrypi/admin/` (do not use the localhost address, use the one specified in the router)

Once you are logged in, navigate to the DNS page under `Settings`:

![pihole-settings- local_dns](https://user-images.githubusercontent.com/123265893/218285445-b98d1e23-0ced-4c18-9821-2099dfd94d3a.png)

Type `127.0.0.1#5335`in the field Custom1 (IPv4).

Now the Raspberry Pi can be restarted.

But one last thing needs to be done: the router needs to know that it is no longer responsible for DNS resolution. This can be achieved by setting the IPv4 address of the local DNS in your router to the IP address of the Raspberry Pi.

To do this click on the button IPv4 Settings in the lower right corner.

![Heimnetz-netzwerk-ip4-ip6_einstellungen-button](https://user-images.githubusercontent.com/123265893/218287228-8c4b4638-ab45-4ca6-ba06-bd89c64b04e1.png)

In the next screen you can enter the IP address:

![Heimnetz-netzwerk-ip4_einstellungen](https://user-images.githubusercontent.com/123265893/218287360-216c02ec-75c8-48c7-ac79-feeecf985de3.PNG)

The localhost address must not be used here either, but the IP address under which the Raspberry Pi can be found in the network must be specified. Leave all other fields untouched.

If you have no internet access after reboot, you can try to provide `::1` in the field Custom 3 (IPv6) on the [DNS settings page](/../../../../TomfromBerlin/Debian-Pihole-Unbound/blob/main/README.md#pi-hole-and-unbound) of Pi-hole. This worked for me. You may notice the little yellow dot in the upper left area. This signals that there might be an issue. Clicking on this will bring you to the following screen:

![pihole-warning](https://user-images.githubusercontent.com/123265893/218285573-443eb6fa-7344-4600-a8a0-a51d4907f54a.png)

This tells us that the IP6 address is somehow redundant since it is the IPv6 address of localhost which is already defined by the IPv4 address (127.0.0.1). So this is ignored by Pi-hole and you can leave it as it is. However, now we should have internet access from every device in our home network and every request should be answered by Unbound and filtered by Pi-hole.

But there's that little, tiny, yellow dot in the top left... And this is where the weird comes in: the IP6 address `::1` can be deleted now and everything still works. Really weird!

When you now search for a website for the first time, it may take a while to be found. The second call and every further one is processed in no time at all.

First search with `dig nasa.gov 127.0.0.1 -p 5335` (note the 'Query time', fourth line from the bottom)):

![dig_nasa_1](https://user-images.githubusercontent.com/123265893/232343572-8fdddceb-88ae-46fc-8d0c-8c43429dfbb1.png)

Second search:

![dig_nasa_2](https://user-images.githubusercontent.com/123265893/232343585-8d034f59-8185-4768-a1a3-a3952a025b48.png)


The fourth line from the top should read `status: NOERROR`. If an error occurs, then something is wrong, obviously. Review all the steps that have been taken. Maybe there's a typo or something. If there is an error in the tutorial, I would be very grateful for information about it. Use the [discussion site](/../../../../TomfromBerlin/Debian-Pihole-Unbound/discussions) for that or report an [issue](/../../../../TomfromBerlin/Debian-Pihole-Unbound/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc).

That's it folks. I hope all of this is of some help.
