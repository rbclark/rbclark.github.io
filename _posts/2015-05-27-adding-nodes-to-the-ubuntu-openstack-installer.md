---
id: 468
title: Adding nodes to the Ubuntu OpenStack Installer
date: 2015-05-27T14:21:35+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=468
permalink: /2015/05/27/adding-nodes-to-the-ubuntu-openstack-installer/
categories:
  - Linux
---
One of the nice things about using Ubuntu to deploy OpenStack is that there are a lot of tools built for the sole purpose easing cloud-deployments with Ubuntu. Unfortunately not all of these tools have the easiest documentation to follow. One of these tools is the Ubuntu OpenStack installer. When you run the installer, by default it creates 3 machines and does not utilize any of the rest of the resources in your maas cluster. You can fix this by using the juju command line tool. On the machine which you ran sudo openstack-install, run the following commands to setup your environment.

> mkdir -p ~/.juju/environments
  
> ln -s ~/.cloud-install/juju/environments.yaml ~/.juju/environments.yaml
  
> ln -s ~/.cloud-install/juju/environments/maas.jenv ~/.juju/environments/maas.jenv

Now you can easily run juju commands without having to set the juju environment. Now in order to add the new nodes, simply run

> juju add-machine machineName.maas
  
> juju add-unit nova-compute &#8211;to NUMBER

Where machineName is the name of the machine commissioned in maas and NUMBER is the number returned from running add-machine.