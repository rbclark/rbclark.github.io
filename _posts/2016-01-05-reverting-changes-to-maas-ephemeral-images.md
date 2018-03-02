---
id: 513
title: Reverting changes to MAAS ephemeral images
date: 2016-01-05T19:26:34+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=513
permalink: /2016/01/05/reverting-changes-to-maas-ephemeral-images/
categories:
  - Metal as a Service
---
In the MAAS &#8220;Debugging ephemeral image&#8221; section, you are provided with steps on how to create a user called &#8220;backdoor&#8221; with the password &#8220;ubuntu&#8221;. Nowhere in these steps does it provide the steps to revert these changes, even though all the first step of the guide is doing is creating a backup. Once you are done with your backdoor, you can revert these changes using the following command.

> imgs=$(echo /var/lib/maas/boot-resources/\*/\*/\*/\*/\*/\*/root-image)
  
> for img in $imgs; do
  
> [ -f &#8220;$img.dist&#8221; ] && sudo mv -f $img.dist $img
  
> done