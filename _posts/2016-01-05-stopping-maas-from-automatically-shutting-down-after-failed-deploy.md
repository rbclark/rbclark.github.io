---
id: 509
title: Stopping MAAS from automatically shutting down after failed deploy
date: 2016-01-05T19:22:28+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=509
permalink: /2016/01/05/stopping-maas-from-automatically-shutting-down-after-failed-deploy/
categories:
  - Metal as a Service
---
I have been attempting to debug an issue involving curtin not successfully <a href="https://code.launchpad.net/~returntoreptar/curtin/removable-drive-fixes/+merge/281707" target="_blank">finding all of the drives</a> on my machine, and during the debugging I ended up formulating a rather reliable way to stop the machine from powering off automatically, as it does whenever deployment fails. Note that this does not require the ephemeral image backdoor listed in the <a href="https://maas.ubuntu.com/docs/troubleshooting.html#debugging-ephemeral-image" target="_blank">MAAS troubleshooting page</a>, however that option may prove more helpful when dealing with errors during commissioning instead of deployment.

In order for this to work you will need to have your machine connected to the MAAS internal network and have your node already commissioned and not yet deployed. In the MAAS UI if you click into a node and click deploy, you can scroll down to the network section where you should see an IP. Paste this IP into the bash line below in place of IP-HERE and run it in your terminal.

> while true; do ssh ubuntu@IP-HERE -oStrictHostKeyChecking=no -oUserKnownHostsFile=/dev/null sudo touch /run/block-curtin-poweroff && break; done

The script will keep trying until the node responds and then will create a file which will stop curtin from powering off your machine. Note that if this does not work you can also try

> touch /tmp/block-poweroff

In place of \`sudo touch /run/block-curtin-poweroff\` in the above bash.