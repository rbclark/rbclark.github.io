---
id: 422
title: 'Enable Wake on LAN: Supermicro H8SGL(-F)'
date: 2015-05-21T07:19:38+00:00
author: Kyle
layout: post
guid: http://wiredtron.com/?p=422
permalink: /2015/05/21/enable-wake-on-lan-supermicro-h8sgl-f/
categories:
  - Other
tags:
  - American Megatrends Inc
  - AMI
  - AMIBIOS
  - H8SGL
  - H8SGL-F
  - Intelligent Platform Management Interface
  - PSSC Labs
  - Supermicro
  - Wake on LAN
  - WoL
---
So you&#8217;re in the process of setting up a sweet Metal as a Service cluster and you&#8217;re taking all the second-hand hardware you can get your hands on. Lucky for you, a relatively beefy unmarked PSSC Labs server (4U) happens to make its way over to your rack.

Nice!

_*Or AMD Equivalent_

Not nice!

With no wake on LAN capability obvious in the BIOS there is no way to enlist in MaaS. The BIOS refers to a Supermicro H8SGL(-F), which I took to mean H8SGL-F. Quick research shows that the -F supports Intelligent Platform Management Interface (IMPI). Flashing updated firmware for the IMPI failed with a &#8220;Send command&#8221; error. At this point it was time to open up the server and check out the motherboard. Inside is the <a href="http://www.supermicro.com/Aplus/motherboard/Opteron6000/SR56x0/H8SGL.cfm" target="_blank">Supermicro H8SGL</a>. Don&#8217;t be confused by the naming convention like I was. The BIOS refers to this motherboard as the H8SGL(-F) because both models utilize the same BIOS. To be certain what motherboard you have you&#8217;ll want to examine the I/O and look for a dedicated IMPI LAN or look for the System Management Bus Header (IPMB) to the left of the internal USB (shown on page 1-4 <a href="http://www.supermicro.com/manuals/motherboard/SR56x0/MNL-H8SGL(-F).pdf" target="_blank">here</a> (PDF)). Both models have the Baseboard Management Controller necessary for IMPI.

Now that we _(finally)_ know our target motherboard, download the most recent BIOS (.zip) from the above link and make a <a href="http://www.freedos.org/wiki/index.php/USB" target="_blank">bootable freeDOS drive</a> and extract the contents of the .zip on the drive. Boot into freeDOS and execute the flash.bat command.

> `flash H8SGL3.B25`

After flashing, manually reboot the computer. When it comes back up it should greet you with a checksum error, so strike F1 to edit settings, strike F9 to load the defaults, and finally F10 to save and exit. The server never restarted and became unresponsive. Wow. The solution was to remove the graphics card and plug into the onboard VGA. After another reboot enter the settings to customize. Plugging the graphics card back in after customization did not cause any issues. Quick note: The BIOS version did not change after the update, but the look & feel is clearly different.

Once you&#8217;re back into the settings you quickly realize nothing has changed and there is still no wake on LAN option. Do not fret! Simply plug into the NIC and send some magic packets its way and the server should roar to life!

We also found out that wake on LAN does not respond after a loss of power. The recommendation is to enable resume on power failure to mitigate this.

This research was done _after_ we had tried throwing wake on lan packets from MaaS at the onboard Intel network interface controller to no avail. Which is strange, because the documentation says the motherboard supports wake on LAN. It seems like a BIOS update was necessary to activate the feature. Wake from PCI 2.2 did not seem to work in a plug-and-play manner. We scavenged an old 10/100Mbps NIC with wake on LAN and sent it wakeup packets before and after the BIOS update. If all else fails there is a wake on LAN 3-pin header for legacy support that can be wired to a NIC with the same 3-pin header.

Obviously I _<del>have had some frustrations with</del> _hate this motherboard, but in the end we have another enlisted node!

* * *

&nbsp;

<div id="attachment_430" style="max-width: 310px" class="wp-caption alignnone">
  <a href="http://wiredtron.com/wp-content/uploads/2015/05/image1.jpg"><img class="wp-image-430 size-medium" src="http://wiredtron.com/wp-content/uploads/2015/05/image1-300x93.jpg" alt="BIOS Output" width="300" height="93" srcset="https://wiredtron.com/wp-content/uploads/2015/05/image1-300x93.jpg 300w, https://wiredtron.com/wp-content/uploads/2015/05/image1.jpg 687w" sizes="(max-width: 300px) 100vw, 300px" /></a>
  
  <p class="wp-caption-text">
    BIOS Output &#8211; H8SGL-F?
  </p>
</div>

<div id="attachment_429" style="max-width: 310px" class="wp-caption alignnone">
  <a href="http://wiredtron.com/wp-content/uploads/2015/05/image2.jpg"><img class="wp-image-429 size-medium" src="http://wiredtron.com/wp-content/uploads/2015/05/image2-300x60.jpg" alt="BMC Error" width="300" height="60" srcset="https://wiredtron.com/wp-content/uploads/2015/05/image2-300x60.jpg 300w, https://wiredtron.com/wp-content/uploads/2015/05/image2.jpg 657w" sizes="(max-width: 300px) 100vw, 300px" /></a>
  
  <p class="wp-caption-text">
    BMC Error
  </p>
</div>

<div id="attachment_426" style="max-width: 310px" class="wp-caption alignnone">
  <a href="http://wiredtron.com/wp-content/uploads/2015/05/image3.jpg"><img class="wp-image-426 size-medium" src="http://wiredtron.com/wp-content/uploads/2015/05/image3-300x123.jpg" alt="AMI BIOS Flash" width="300" height="123" srcset="https://wiredtron.com/wp-content/uploads/2015/05/image3-300x123.jpg 300w, https://wiredtron.com/wp-content/uploads/2015/05/image3-1024x420.jpg 1024w, https://wiredtron.com/wp-content/uploads/2015/05/image3.jpg 1106w" sizes="(max-width: 300px) 100vw, 300px" /></a>
  
  <p class="wp-caption-text">
    Flash command execution and output
  </p>
</div>

* * *

&nbsp;