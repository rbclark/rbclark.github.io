---
id: 439
title: Reviving Quantum Guardian 4400 NAS
date: 2015-05-22T08:04:29+00:00
author: Kyle
layout: post
guid: http://wiredtron.com/?p=439
permalink: /2015/05/22/reviving-quantum-guardian-4400-nas/
categories:
  - Linux
tags:
  - GuardianOS
  - NAS
  - Network Attached Storage
  - Quantum Guardian 4400
  - Snap Server 4400
---
In keeping with the [previous post](http://wiredtron.com/2015/05/21/enable-wake-on-lan-supermicro-h8sgl-f/) about getting second-hand hardware I present the Quantum Guardian 4400! Once a [state of the art](http://www.pcmag.com/article2/0,2817,671848,00.asp) machine, the Guardian 4400 is now a shadow of its former self. It is vulnerable to a host of vulnerabilities, most notably [shell-shock](http://en.wikipedia.org/wiki/Shellshock_(software_bug)), The icing on the cake is that to (officially) patch the vulnerabilities it will cost you approx. $100 for their GuardianOS. Other options include installing a different, up-to-date version of Linux. Not to mention that it takes half a day to rebuilt the RAID (_its sooo speedy). _This unit was also known as the Snap 4400.

So on to the revival&#8230;

In the context of this post, reviving really means wiping the settings, data, hooking it up to a network that will utilize it, and generally not neglect it as much as it has been in the past. People who want to utilize some secondhand 4400 hardware with minimal effort, this post is for you!

Since there was no data on the 4400 that was useful a full wipe was in order! If you can get the administrator password from the previous owner it may save you a bit of time. If not, get a paperclip or a pen and hold the reset button to the right of the NIC while you press the power button. This will boot the 4400 into recovery mode where you&#8217;ll have the option to do a full settings reset.

The next step is to find the machine on the network. If the 4400 was configured with a static IP before do not worry &#8211; it seems to ignore that setting. I found out the IP address of the 4400 by checking the ARP table in the router. I copy and pasted suspect IPs to the web browser and got to the web UI of the 4400. The web UI works O.K. in Safari, but better in Chrome. In the webUI select the full reset option and reboot. Note that this full reset will not erase drive data. When the system comes back online, go back to the web UI and &#8220;Administer&#8221; with the default username: admin and password: admin.

Erase the RAID by going into Storage → RAID Sets → Click delete and select the RAID to delete. Build a a new RAID in the same RAID Set menu via the Create RAID Set button. This may take a while and the 4400 web interface may become unresponsive during it. Another cause for the webUI is faulty disks. The web UI will display any disks that have errors, additionally a drive health light at the front of the machine will be amber. I discarded the drives that threw an error. After the RAID set is built you can select what type of connections to accept, permissions, and quotas. The manual is very helpful for all of that.

On to networking&#8230; In my experience with the 4400 the static setting does nothing and is overwritten by your DHCP server. Better to set it there. The networking section does give you the MAC of both NICs. I went with a standard networking configuration.

One 4400 refused to boot. I supposed it was an operating system error because of the system health light. It will alternate green and amber when when the 4400 is powered on. The COM port is the way to debug this, and I never got that working well enough to see the startup messages and view the errors. Besides, putting a new OS on it would&#8217;ve been out of my price range.

Some cool projects others have done to modernize their 4400 is flash a better OS, and convert the NAS from PATA to SATA.

&nbsp;