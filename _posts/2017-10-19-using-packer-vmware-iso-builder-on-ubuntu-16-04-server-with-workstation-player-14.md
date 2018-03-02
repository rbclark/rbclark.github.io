---
id: 571
title: Using Packer vmware-iso builder on Ubuntu 16.04 server with Workstation Player 14
date: 2017-10-19T15:59:42+00:00
author: Robert
layout: post
guid: https://wiredtron.com/?p=571
permalink: /2017/10/19/using-packer-vmware-iso-builder-on-ubuntu-16-04-server-with-workstation-player-14/
categories:
  - Linux
  - Open Source Programs
---
Gwetting VMWare Workstation Player 14 (the free one) to work with Packer on Ubuntu 16.04 is possible and not very difficult, however it is very undocumented. In order to get it working you will need a few different packages, listed below

  1. [Packer](https://www.packer.io/downloads.html) (Download the Linux 64-bit version, at the time of this guide the latest version is 1.1.1)
  2. [VMWare Workstation Player 14](https://my.vmware.com/en/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/14_0) (I downloaded the version for Linux 64-bit from the previous link)
  3. [VMWare Vix 1.17](https://my.vmware.com/en/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/14_0%7CPLAYER-1400%7Cdrivers_tools) (I downloaded the version for Linux 64-bit from the previous link)
  4. In order to build kernel modules you will need to <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">sudo apt install libxt6 libxtst6 libxcursor-dev libxinerama-dev build-essential</code>
  5. Install additional dependencies by running <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">sudo apt install qemu-utils libxi6</code>

Install packer by downloading the zip and moving the packer executable into /usr/local/bin. Next, download the VMWare Workstation Player 14 bundle and install it by running <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">sudo sh VMWare-Player-14.bundle</code> . Finally, download the VMWare Vix bundle and install it by running  <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">sudo sh VMWare-Vix.bundle</code> .

Next you will need to run <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">sudo vmware-modconfig --console --install-all</code> to install all of the required kernel modules.

You should now be able to do packer builds using the vmware-iso builder on your headless machine running Ubuntu 16.04.

If the execution fails here with an error such as &#8220;Error: The operation was canceled&#8221;, then run packer with the command <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">PACKER_LOG=1 packer build -debug &lt;yourpackerfile&gt;.json</code> which will allow you to step through each step of the packer build and pause before cleanup to inspect the logs written to your output directory, which is normally <code class="prettyprint lang-sh" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">output-&lt;vmname&gt;</code>. Inside that folder will be a vmware log file which should give you some hints towards what is going wrong.

Credits:

  *  https://kradalby.no/setup-vmware-player-headless-on-debian.html got me the final undocumented step required to setup vmware-modconfig kernel modules.
