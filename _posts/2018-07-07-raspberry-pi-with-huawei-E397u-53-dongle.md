---
id: 572
title: Setting up Raspberry PI with Huawei E397u-53 dongle on Metro PCS
date: '2018-07-07 00:00:00 -0500'
author: Robert
layout: post
guid: https://wiredtron.com/?p=572
permalink: /2018/07/07/raspberry-pi-with-huawei-E397u-53-dongle-metro/
categories:
  - Linux
  - Open Source Programs
---
The Huawei E397u-53 dongle is a low cost dongle which is available on Amazon for around $20. It is possible to get it working on startup on a Raspberry PI, and there is some very good documentation which I will mention in the credits of this post. Even with the documntation I was able to find, I still hit a decent number of roadblocks during setup.

First up, I am going to summarize the actual steps I used, which are mostly the same as the one from the first blog post. I opted to use the NDISDUP method since WvDial is slower and harder to reliably bring up on startup.

Also as a note, most blog posts mention needing to mess with usb_modeswitch. As of the time of this posting, with Raspbian 9 (Stretch) on a Raspberry PI 3 Model B version 1.2 this was not necessary since the device already showed up on `/dev/ttyUSB0`

1. Install screen using `sudo apt-get install screen` and then run `screen /dev/ttyUSB0`
2. This will put you in a console that will look hung. If you type `AT` and hit enter you should get a response of OK back.
3. In the new console, type `AT^NDISDUP=1,1,"<APN>"` where \<APN\> is the APN for your carrier. In my case it was `fast.metropcs.com`.
4. You will now need to install libqmi-utils. Unfortunately this is where I ran into the biggest issue. The current version of libqmi-utils in Rasbian has a bug (see 2nd link in credits) where you will get an error stating:
```
sudo APN=fast.metropcs.com /usr/bin/qmi-network /dev/cdc-wdm0 start
Loading profile at /etc/qmi-network.conf...
    APN: fast.metropcs.com
    APN user: unset
    APN password: unset
    qmi-proxy: no
Checking data format with 'qmicli -d /dev/cdc-wdm0 --wda-get-data-format '...
error: couldn't create client for the 'wda' service: QMI protocol error (31): 'InvalidServiceType'
Device link layer protocol not retrieved: WDA unsupported
Starting network with 'qmicli -d /dev/cdc-wdm0 --wds-start-network=apn='fast.metropcs.com'  --client-no-release-cid '...
error: couldn't start network: QMI protocol error (64): '(null)'
Saving state at /tmp/qmi-network-state-cdc-wdm0... (CID: 1)
error: network start failed, no packet data handle
Clearing state at /tmp/qmi-network-state-cdc-wdm0...
```
5. In order to workaround this, I first removed the version of libqmi-utils I installed by running `sudo apt remove libqmi-utils` and then reinstalled it from source by running
```
wget http://www.freedesktop.org/software/libqmi/libqmi-1.16.0.tar.xz
tar -vxf libqmi-1.16.0.tar.xz
cd libqmi-1.16.0
sudo apt-get install glib-networking libmbim-glib-dev libgudev-1.0-dev
./configure --prefix=/usr --disable-static
make && sudo make install
sudo qmi-network /dev/cdc-wdm0 start
```
6. Once I was able to verify that `sudo qmi-network /dev/cdc-wdm0 start` worked, I went ahead and rebooted the machine. From there I then edited `/etc/network/interfaces.d/wwan0`, replacing \<APN\> with my APN settings and added
```
allow-hotplug wwan0
iface wwan0 inet dhcp
     pre-up for _ in $(seq 1 10); do /usr/bin/test -c /dev/cdc-wdm0 && break; /bin/sleep 1; done
     pre-up for _ in $(seq 1 10); do /usr/bin/qmicli -d /dev/cdc-wdm0 --nas-get-signal-strength && break; /bin/sleep 1; done
     pre-up APN=<APN> /usr/bin/qmi-network /dev/cdc-wdm0 start
     post-down /usr/bin/qmi-network /dev/cdc-wdm0 stop
```
7. I was then able to reboot the PI and the networking came up successfully.
8. At this point everything was working, however I did not have any DNS resolution. In order to fix this I had to edit `/etc/resolv.conf.head` and add
```
#OpenDns Servers
nameserver 1.1.1.1
nameserver 1.0.0.1
```

Credits:
  * https://robpol86.com/raspberry_pi_project_fi.html This article got me about 90% there, the biggest issue being the current version of libqmi-utils being broken.
  * https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=845979 This bug report tipped me off to roll the libqmi-utils version back to before 1.16.2.
  * https://gist.github.com/alvaro893/f5b8dd026eb98959e8c6c59a45b63cc2
