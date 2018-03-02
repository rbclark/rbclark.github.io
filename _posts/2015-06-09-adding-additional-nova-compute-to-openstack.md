---
id: 478
title: Adding Additional Nova-Compute to OpenStack
date: 2015-06-09T14:25:11+00:00
author: Kyle
layout: post
guid: http://wiredtron.com/?p=478
permalink: /2015/06/09/adding-additional-nova-compute-to-openstack/
categories:
  - Juju
  - Linux
  - Open Source Programs
  - OpenStack
tags:
  - Juju
  - MaaS
  - Metal as a Service
  - nova
  - nova-compute
  - OpenStack
  - openstack-installer
---
From openstack-installer base we want to add additional nova-compute services for different types of virtualization. The setup we have keeps one hypervisor type on one machine. This means that we have one machine dedicated KVM, another dedicated LXC, etc. When we want/need more of a particular hypervisor, just tell Juju and it will create a new machine with the same config as the first charm.

OpenStack starts you off with a single nova-compute of virtualization KVM. To begin, set Juju environment as described in a [previous](http://wiredtron.com/2015/05/27/adding-nodes-to-the-ubuntu-openstack-installer/) post on the MaaS server. Use Juju to create another nova-compute charm, edit the configuration, and create the relations.

> juju deploy &#8211;to 4 nova-compute nova-compute-lxc
> 
> juju set nova-compute-lxc openstack-origin=cloud:trusty-kilo
> 
> juju add-relation nova-compute-lxc nova-cloud-controller
> 
> juju add-relation nova-compute-lxc glance
> 
> juju add-relation nova-compute-lxc rabbitmq-server
> 
> juju add-relation nova-compute-lxc mysql
> 
> juju add-relation nova-compute-lxc ntp
> 
> juju add-relation nova-compute-lxc neutron-openvswitch

After the setup you can edit the configuration (if supported) through Juju, or you can edit nova&#8217;s configuration manually if it is not supported by the charm (like [Docker](http://wiredtron.com/2015/06/09/docker-in-kilo/)).