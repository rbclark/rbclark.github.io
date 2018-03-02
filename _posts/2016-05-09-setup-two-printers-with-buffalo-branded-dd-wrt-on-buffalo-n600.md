---
id: 537
title: Setup two Printers with Buffalo Branded DD-WRT on Buffalo N600
date: 2016-05-09T17:09:48+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=537
permalink: /2016/05/09/setup-two-printers-with-buffalo-branded-dd-wrt-on-buffalo-n600/
categories:
  - Uncategorized
tags:
  - Buffalo DD-WRT
  - USB Printing Support
---
The custom version of DD-WRT offered by Buffalo makes it easy to setup one printer with your router, however it becomes much harder when you go to add a second one. All the guides currently online (with the exception of this one) fail to cover how to do it. First off, you must make sure in your router control panel that you have &#8220;USB Printer Support&#8221; is turned on (located under Services > USB). Next you will need to connect your two printers through a USB hub to the router. Finally you will need to navigate to Administration > Commands and type the command \`p910nd -f /dev/lp1 1 -t 5\` and then choose &#8220;Save Startup&#8221;. You can then reboot the router and should have the ability to access both of your printers if you follow the instructions located [here](https://www.bestvpn.com/blog/8927/sharing-your-printer-with-dd-wrt/).

<a href="http://wiredtron.com/2016/05/09/setup-two-printers-with-buffalo-branded-dd-wrt-on-buffalo-n600/screen-shot-2016-05-09-at-5-03-28-pm/" rel="attachment wp-att-538"><img class="size-medium wp-image-538 alignnone" src="http://wiredtron.com/wp-content/uploads/2016/05/Screen-Shot-2016-05-09-at-5.03.28-PM-300x216.png" alt="Screen Shot 2016-05-09 at 5.03.28 PM" width="300" height="216" srcset="https://wiredtron.com/wp-content/uploads/2016/05/Screen-Shot-2016-05-09-at-5.03.28-PM-300x216.png 300w, https://wiredtron.com/wp-content/uploads/2016/05/Screen-Shot-2016-05-09-at-5.03.28-PM.png 618w" sizes="(max-width: 300px) 100vw, 300px" /><br /> </a>The reason for this is that by default when you enable USB Printer Support, it only runs the command for the first printer (you can see this if you run the `ps` command on the router and will see something like &#8220;{p910nd} p9100d -f /dev/lp0 0 -t 5&#8221; as one of the running services. For more information about p910nd you can find it [here](http://p910nd.sourceforge.net/).