---
title: Raspberry pi headless
tags: [wiki]
categories: wiki
---

Comment configurer le ssh sur une raspberry pi 0 ?

```
$ cd /run/media/thomas/xx/boot
$ touch ssh
$ vim wpa_supplicant.conf
	country=FR
	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	update_config=1

	network={
			ssid="NETWORK-NAME"
			psk="NETWORK-PASSWORD"
	}
```

Eject SD card
Plug in raspberry and power on
Wait 30-90sec
```
$ nmap -sn 192.168.x.0/24
$ ssh pi@192.168.x.x
```