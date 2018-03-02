---
id: 516
title: Modifying MAAS deployment library scripts directly (such as curtin)
date: 2016-01-06T09:49:21+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=516
permalink: /2016/01/06/modifying-maas-deployment-library-scripts-directly-such-as-curtin/
categories:
  - Metal as a Service
---
Say you have found a bug in a library which MAAS uses for deploying servers, have already fixed the bug and put in a merge request, however you need that change in your MAAS environment now. In my case it was a bug in curtin which was causing half of my machines to not deploy. The good news is that you can modify the curtin code directly on your MAAS node, working around the issue until your merge request is accepted. In my case the library was located on the MAAS node at

> <p class="p1">
>   <span class="s1">/usr/lib/python2.7/dist-packages/curtin</span>
> </p>

<p class="p1">
  However if you are dealing with a different package then the dist-package directly will most likely contain it. The mainline code for curtin is located <a href="https://code.launchpad.net/curtin" target="_blank">here</a>, so any changes you need can easily be copy and pasted into the curtin code directly at the path listed above and it will then be copied to the remote machines and used for orchestrating deployment.
</p>