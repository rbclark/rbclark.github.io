---
id: 543
title: Setting up MaaS + Juju on one Node on Ubuntu (using Virsh)
date: 2016-08-18T12:17:30+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=543
permalink: /2016/08/18/setting-up-maas-juju-on-one-node-on-ubuntu-using-virsh/
categories:
  - Uncategorized
---
Using MaaS and Juju to deploy Openstack on Ubuntu is pretty easy to do, however often requires a large number of machine sitting around in order to host everything. Fortunately it is possible to run your Juju controller alongside your MaaS machine using virsh. For the purpose of this guide we will be using MaaS 2.0 and Juju 2.0.

First you will need to install MaaS and get it working. In my case I have 2 interfaces, my private interface which is managed by MaaS and my public interface which is managed by my router. My initial config looked as follows:

> \# The loopback network interface
>
> auto lo
>
> iface lo inet loopback
>
> \# MaaS network interface
>
> auto eth0
>
> iface eth0 inet static
>
> address 172.16.0.1
>
> netmask 255.255.0.0
>
> \# The primary network interface
>
> auto eth1
>
> iface eth1 inet dhcp

This config worked fine for commissioning machines inside MaaS, however it failed when I went to create a virtual machine on my MaaS node to run Juju. In order to get virtual machines working I had to do the following:

Install virsh and other required packages:

> <p class="p1">
>   <span class="s1">sudo apt-get -y </span><span class="s2">install</span><span class="s1"> libvirt-bin </span><span class="s1">linux-image-extra-virtual </span><span class="s1">kvm virt-manager</span>
> </p>

<p class="p1">
  Add the MaaS user to the libvirtd group:
</p>

> <p class="p1">
>   <span class="s1">sudo </span><span class="s2">usermod</span><span class="s1"> -G libvirtd -a maas</span>
> </p>

<p class="p1">
  Update my networking config to look as follows:
</p>

> <p class="p1">
>   <span class="s1"># The loopback network interface<br /> </span><span class="s1">auto lo<br /> </span><span class="s1">iface lo inet loopback<br /> </span><span class="s1">iface enp9s0f0 inet manual</span>
> </p>
>
> <p class="p1">
>   <span class="s1"># The primary network interface<br /> </span><span class="s1">auto enp9s0f1<br /> </span><span class="s1">iface enp9s0f1 inet dhcp<br /> </span><span class="s1">auto br0<br /> </span><span class="s1">iface br0 inet static<br /> </span><span class="s1"><span class="Apple-converted-space">    </span>bridge_ports enp9s0f0<br /> </span><span class="s1"><span class="Apple-converted-space">    </span>address 172.16.0.1<br /> </span><span class="s1"><span class="Apple-converted-space">    </span>netmask 255.255.0.0</span>
> </p>

<p class="p1">
  After updating your config, <strong>reboot the MaaS machine!</strong> This step is very important as networking will not work until this step is completed.
</p>

<p class="p1">
  Finally, I was then able to spin up a virtual machine using the following command:
</p>

> <p class="p1">
>   virt-install \<br /> &#8211;name Juju-Controller-Node \<br /> &#8211;ram 8192 \<br /> &#8211;disk path=/var/kvm/images/Juju-Controller-Node.img,size=150 \<br /> &#8211;network=bridge:br0 \<br /> &#8211;vcpus 4 \<br /> &#8211;os-type linux \<br /> &#8211;os-variant ubuntu16.04 \<br /> &#8211;graphics none \<br /> &#8211;pxe \<br /> &#8211;accelerate \<br /> &#8211;boot network
> </p>

<p class="p1">
  Success! The node then showed up in MaaS. I then configured the power settings on the newly added machine to look as follows:
</p>

<p class="p1">
  <a href="http://wiredtron.com/2016/08/18/setting-up-maas-juju-on-one-node-on-ubuntu-using-virsh/screen-shot-2016-08-18-at-12-15-06-pm/" rel="attachment wp-att-544"><img class="size-medium wp-image-544 alignnone" src="http://wiredtron.com/wp-content/uploads/2016/08/Screen-Shot-2016-08-18-at-12.15.06-PM-300x152.png" alt="Screen Shot 2016-08-18 at 12.15.06 PM" width="300" height="152" srcset="https://wiredtron.com/wp-content/uploads/2016/08/Screen-Shot-2016-08-18-at-12.15.06-PM-300x152.png 300w, https://wiredtron.com/wp-content/uploads/2016/08/Screen-Shot-2016-08-18-at-12.15.06-PM-768x388.png 768w, https://wiredtron.com/wp-content/uploads/2016/08/Screen-Shot-2016-08-18-at-12.15.06-PM-1024x518.png 1024w, https://wiredtron.com/wp-content/uploads/2016/08/Screen-Shot-2016-08-18-at-12.15.06-PM.png 1116w" sizes="(max-width: 300px) 100vw, 300px" /><br /> </a>I was then able to commission and deploy the node successfully.
</p>
