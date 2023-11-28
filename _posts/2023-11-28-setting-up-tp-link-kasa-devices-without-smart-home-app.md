---
title: Setting up TP Link Kasa Devices without Smart Home app
---

Kasa plugs and switches are some of the best smart home devices on the market and have been for many years. One of their most attractive attributes is the existence of a local LAN API that can be interacted with. Even though this use is not officially supported by Kasa, in recent years they have gone from actively [blocking local access](https://forum.universal-devices.com/topic/33115-kasa-will-no-longer-allow-local-access/) to more or less accepting it and even[ fixing bugs ](https://community.tp-link.com/en/smart-home/forum/topic/638558)that arise when users use their switches exclusively offline. For me one of the biggest annoyances was manually having to do setup and configuration through their app, so I did some digging to see if there was a better way, and luckily there is! In this post I will walk you through the steps to take a Kasa smart plug from out of the box to fully configured. This was tested using a HS103 running firmware 1.0.6 (which is what is came with from the faactory) but I have used these same commands against HS220 and other switches as well and they have all worked.

A big thanks to softScheck for posting a full list of commands on [Github](https://github.com/softScheck/tplink-smartplug/blob/master/tplink-smarthome-commands.txt).

### Setup the TPLink CLI

In order to use these commands you will need a somewhat recent version of `node`. Once you have node, you will need to install the [tplink-smarthome-api](https://github.com/plasticrake/tplink-smarthome-api) using `npm install -g tplink-smarthome-api`. This API has a lot of commands available, all of which can be listed by running `tplink-smarthome-api --help`.

### Connect to the Device

When the device is plugged in it automatically creates a network containing the last 4 of it's MAC address at the end. This network is unsecured so you can just connect to it with your computer. In my testing, the devices always come up with the IP `192.168.0.1` but you can verify this information by running `ifconfig` (or `ipconfig` on Windows). Once you are on the devices network you can finalize setup.

### Setup the Device

1. Use the TPLink smarthome CLI to query the device by running `tplink-smarthome-api getInfo 192.168.0.1`. This information includes the MAC address which I use to assign a static IP in my router. 
2. (Optional) Before connecting it to your network you can give it a name by running `tplink-smarthome-api setAlias 192.168.0.1 'Device Name'`.
3. Run `tplink-smarthome-api send 192.168.0.1 '{"netif":{"set_stainfo":{"ssid":"<ssid>","password":"<password>","key_type":3}}}'` to set your WIFI. This will cause the device to automatically disconnect and join the specified WIFI network.

From there you can jump back over to your WIFI network and send commands directly to the device at it's IP address on your network. For example, to turn the device on/off run `tplink-smarthome-api setPowerState <new-ip> 1` to turn the device on.

And that's it, enjoy managing your devices without an app!
