---
id: 573
title: Expanding LVM Storage with Proxmox 5.2
date: '2018-07-28 00:00:00 -0500'
author: Robert
layout: post
guid: https://wiredtron.com/?p=573
permalink: /2018/07/07/extending-lvm-with-proxmox-52/
categories:
  - Linux
  - Open Source Programs
---
Proxmox is an Open Source virtualization solution that provides very easy installation at an irresistible price point. The biggest issue I have encountered so far after messing with it for 2 days is its lack of customization for installing across multiple drives. My initial goal was to have a setup that I have accomplished before with ESXi where I have a main drive and a secondary storage drive. In this setup, the main OS drive contains my OS and the secondary drive stores the VMs. Unfortunately, after the installation completed I found myself with both the root and data stores built on the main drives. This meant that I didn't have enough space to store my ISOs or VMs. In order to fix this I needed to extend both the data and root volumes. Fortunately this is not my first go with LVM. The steps I used are as follows:

1. Open up fdisk for the secondary drive, in my case this was `fdisk /dev/sdb`. You will need to choose `d` to delete any partitions on the disk and then `w` to write changes.
1. Run `sgdisk -gN 1 /dev/sdb` in order to convert this disk to GPT format. the `-g` option here even if If sgdisk finds a valid MBR or BSD disklabel but no GPT data.
1. `pvcreate /dev/sdb1` initializes Physical Volume for later use by the Logical Volume Manager.
1. `vgextend pve /dev/sdb1` extends the `pve` volume group (which all 3 Proxmox volumes are a part of) to include the newly allocated Physical Volume.
1. This step can be adjusted as needed. For my use case I decided extending my root volume by 80g was sufficient, so I ran `lvresize -L +80g --resizefs /dev/pve/root`. If you need more space, go ahead and change that 80g to something else.
1. I then allocated the rest of my space to the data volume for my VMs, using the command `lvresize -l +100%FREE --resizefs /dev/pve/data`.
