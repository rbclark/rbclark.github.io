---
id: 413
title: Updating the RAID controller on x3650 M2 (7947)
date: 2015-05-19T16:21:41+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=413
permalink: /2015/05/19/updating-the-raid-controller-on-x3650-m2-7947/
categories:
  - Linux
tags:
  - drivers
  - IBM
  - Linux
  - LSI
  - RAID
  - SAS
  - System X
  - Ubuntu
  - UEFI
---
I recently began working with an IBM System X 3650 M2 (7947) with no operating system installed and a very out of date MR10i SAS RAID controller. Unfortunately I was caught in a bit of a catch 22 since I had no OS installed and could not figure out how to get into the RAID controller to setup the drives since it was not showing up in UEFI. Everything I read online stated that it could be found in UEFI at the path: System Settings → Adapters and UEFI drivers → Please refresh this page on the first visit → LSI Controller, however whenever I went there I could only see additional NICs, not the controller. In order to fix this I created a bootable Ubuntu 14.04 64 bit Desktop USB with persistence (steps for this can be found [here](http://www.psychocats.net/ubuntu/usb)). Once I had a bootable USB stick I booted the server and hit <F12> when the option displayed and booted from USB. Once Ubuntu booted I opened the browser and downloaded the RAID update .bin file from IBM which can be found [here](https://www-947.ibm.com/support/entry/portal/docdisplay?lndocid=MIGR-5082023). I then had to install the following packages in order for the updater to run.

> sudo apt-get install -y lib32ncurses5 lib32stdc++6

The first package adds support for running 32 bit binaries (such as the RAID updater) on a 64 bit system. The second allows the updater to find the RAID controller and update it. Once these two packages are installed, you can run the firmware update you have downloaded with the following commands:

> chmod +x ibm\_fw\_sraidmr\_10i-10is-11.0.1-0042\_linux_32-64.bin
  
> ./ibm\_fw\_sraidmr\_10i-10is-11.0.1-0042\_linux_32-64.bin -s

After you reboot the update will be applied and the RAID option will now appear in UEFI under the aforementioned path.