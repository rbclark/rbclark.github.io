---
id: 472
title: MaaS Timeout Error
date: 2015-05-29T07:40:26+00:00
author: Kyle
layout: post
guid: http://wiredtron.com/?p=472
permalink: /2015/05/29/maas-timeout-error/
categories:
  - Linux
tags:
  - firewall
  - IPv4
  - IPv6
  - MaaS
  - Metal as a Service
---
When setting up MaaS there was an odd timeout error when commissioning machines. The PXE image was not able to run `apt` and was timing out. Turns out the firewall and Intrusion Detection System (IDS) was blocking IPv6 traffic to Ubuntu archives even though the rules were configured to allow. The firewall did not seem to block the IPv4 traffic. The cool/strange part was it was an intermittent issue because when `apt` timed out, it would switch archive servers in the middle of the download &#8211; awesome! The solution was to explicitly allow the IPv6/IPv4 traffic from the MaaS server. Now new machines commission normally.