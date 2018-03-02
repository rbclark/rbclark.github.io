---
id: 480
title: Docker in Kilo
date: 2015-06-09T07:52:12+00:00
author: Kyle
layout: post
guid: http://wiredtron.com/?p=480
permalink: /2015/06/09/docker-in-kilo/
categories:
  - Open Source Programs
tags:
  - docker
  - glance
  - kilo
  - nova
  - nova-compute
  - nova-docker
  - OpenStack
---
I have followed [these](https://wiki.openstack.org/wiki/Docker) instructions for installing the Docker driver for OpenStack Kilo &#8211; Canonical distribution. Seems like that document is the most referenced and most up-to-date source. The goal here is to document the ways in which the instructions differed with a working Docker on Kilo setup. Keeping in mind that I&#8217;m following that document rather strictly, starting at the &#8220;Configure OpenStack to Enable Docker&#8221;.

**Starting from scratch:**

The first thing is to install Docker following the [Ubuntu Linux instructions](http://docs.docker.com/installation/ubuntulinux/). If you have a working Docker installation on your nova-compute host this can be skipped. Remember, I am using the Canonical distribution of OpenStack and my nova-compute service, code-name nova-compute-docker, is running 14.04 LTS &#8220;Trusty&#8221;. Connect to your nova-compute-docker host from MaaS using `juju ssh [options]`. See [previous](http://wiredtron.com/2015/05/27/adding-nodes-to-the-ubuntu-openstack-installer/) posts for instruction on how to set that up. I followed the basic install Docker instructions and then created the Docker group. Everything installed and configured without issue.

**Back to the original instructions&#8230;**

The service is no longer called `openstack-nova-compute` in Kilo, instead restart the service with `service nova-compute restart` to have the changes in groups take effect.

The `pip install -e git+<a class="external free" href="https://github.com/stackforge/nova-docker#egg=novadocker" rel="nofollow">https://github.com/stackforge/nova-docker#egg=novadocker</a>` command will work, but executing `python setup.py install` will fail with an &#8220;IncompleteRead&#8221;. Fixing pip is as easy as removing the Ubuntu pip package and upgrading python&#8217;s pip. `sudo apt-get remove python-pip` and `sudo easy_install -U pip`. Afterwards `setup.py install` was successful. Several unsuccessful setup.py runs did not negatively impact the installation as far as I know.

Instead of editing nova.conf, you must edit nova-compute.conf for the docker virt driver in Kilo. Both flags in the file must be changed. `compute_driver=novadocker.virt.docker.DockerDriver` and `virt_type=docker`.

The following information was gained from [this](https://bugs.launchpad.net/nova/+bug/1461217) post on launchpad.

Edit `/usr/lib/python2.7/dist-packages/nova/compute/hv_type.py` to include a `virt_type` definition for docker

`DOCKER="docker"` and add Docker to the list of virt types as shown in the launchpad post. `... ZVM, DOCKER)`

Connect to your glance host from MaaS using `juju ssh [options]`. Glance configuration is done in the `/etc/glance/glance-api.conf` configuration file. I did not see see the `container_format` option but just add it under `[default]` heading. I used the same line from the wiki documentation: `container_formats = ami,ari,aki,bare,ovf,docker`. Restart the Glance API service with `service glance-api restart` to save the `container_formats` change.

Be careful about **restarting** machines without supported flags/configuration options. Juju will overwrite the configuration to match. I asked on IRC (#juju), and found out that implementation of not overriding specific configuration after deployment is charm specific. I am currently looking for a way (hack) to prevent this from happening. 

**Edit: **Hack found: Make the glance-api.conf file immutable `sudo chattr +i /etc/glance/glance-api.conf`.

**Running a container:**

First, `docker pull` the container you wish to run. I started off simple with ubuntu:15.04. After the pull completes, run the following from nova-compute-docker using OpenStack admin-level credentials.

`docker save ubuntu:15.04 | glance image-create --is-public=True  --container-format=docker --disk-format=raw --name ubuntu:15.04`

**Note this important note from the wiki instructions: &#8220;NOTE: The name you provide to glance must match the name by which the image is known to docker.&#8221;** So the string that comes after `save` needs to be the same as the string which comes after `--name`.

Launching the ubuntu:15.04 image from the Horizon dashboard was successful, but the container exits immediately. This causes an error, which is fine because the container really was not told to do anything. Make sure to run containers which have correct ENTRYPOINT/CMD.

Overall this was pretty easy. By far the easiest thing to complete with this whole OpenStack deployment.

Also this, <https://youtu.be/hlsepLGHx8E?t=20m28s>

lol

**Edit: **It seems as if the Docker in Docker scenario doesn&#8217;t work because there is no way to edit the run-time configuration the run command to include the necessary &#8211;privileged flag.