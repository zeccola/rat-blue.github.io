---
title: Hacking my Wood Pellet Smoker
description: Trying my hand at reverse engineering an undocumented API
date: 2024-04-22T00:40:04+10:00
draft: true
#tags: [tag names] (optional)
---


I recently bought myself a [Pit Boss 850 Pro](https://au.pitboss-grills.com/grills/wood-pellet/pit-boss-pro-850-wood-pellet-smoker-with-cover) - I like cooking and fairly recently got into smoking meats. I found that my model actually supports wi-fi, which was pretty attractive to me since I knew there had to be a way to hack it. There is an official app that you can use to connect it to wi-fi, which I did, but the app is kind of garbage (2.5 stars on the [Google Play Store](https://play.google.com/store/apps/details?id=com.pitbossgrills.app)).


# Initial investigation
Initially my investigation started by investigating the local device's IP. A quick port scan revealed that HTTP was open, running Mongoose/6.16.

```
ethan@wsl:nmap -p- 10.0.0.145
Starting Nmap 7.80 ( https://nmap.org ) at 2025-04-10 20:31 AEST
Nmap scan report for 10.0.0.145
Host is up (0.015s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http
```

Once we browse to it's webpage, we see a bunch of files, which appear to be Javascript, certificate and json files that the smoker uses:

| Name              | Modified           | Size  |
|-------------------|--------------------|-------|
| api_dataview.js   | 01-Jan-1970 00:00  | 7.3k  |
| ca.pem            | 01-Jan-1970 00:00  | 6.1k  |
| api_uart.js       | 01-Jan-1970 00:00  | 5.6k  |
| api_net.js        | 01-Jan-1970 00:00  | 5.0k  |
| api_i2c.js        | 01-Jan-1970 00:00  | 4.8k  |
| lib_ws.js         | 01-Jan-1970 00:00  | 4.7k  |
| app.js            | 01-Jan-1970 00:00  | 4.0k  |
| api_events.js     | 01-Jan-1970 00:00  | 3.8k  |
| api_http.js       | 01-Jan-1970 00:00  | 3.7k  |
| api_aws.js        | 01-Jan-1970 00:00  | 3.7k  |
| api_gpio.js       | 01-Jan-1970 00:00  | 3.5k  |
| api_rpc.js        | 01-Jan-1970 00:00  | 2.9k  |
| api_mqtt.js       | 01-Jan-1970 00:00  | 2.9k  |
| lib_http.js       | 01-Jan-1970 00:00  | 2.7k  |
| api_bt_gatts.js   | 01-Jan-1970 00:00  | 2.3k  |
| api_config.js     | 01-Jan-1970 00:00  | 2.1k  |
| api_file.js       | 01-Jan-1970 00:00  | 2.1k  |
| api_log.js        | 01-Jan-1970 00:00  | 2.0k  |
| api_math.js       | 01-Jan-1970 00:00  | 1.8k  |
| api_timer.js      | 01-Jan-1970 00:00  | 1.7k  |
| api_shadow.js     | 01-Jan-1970 00:00  | 1.6k  |
| api_sys.js        | 01-Jan-1970 00:00  | 1.6k  |
| polyfill.js       | 01-Jan-1970 00:00  | 1.4k  |
| api_dht.js        | 01-Jan-1970 00:00  | 1.4k  |
| api_bt_gattc.js   | 01-Jan-1970 00:00  | 1.3k  |
| api_arch_uart.js  | 01-Jan-1970 00:00  | 1.2k  |
| api_wifi.js       | 01-Jan-1970 00:00  | 1020  |
| api_bt_gatt.js    | 01-Jan-1970 00:00  | 886   |
| api_bitbang.js    | 01-Jan-1970 00:00  | 815   |
| api_adc.js        | 01-Jan-1970 00:00  | 803   |
| api_pwm.js        | 01-Jan-1970 00:00  | 726   |
| api_dash.js       | 01-Jan-1970 00:00  | 703   |
| api_bt_gap.js     | 01-Jan-1970 00:00  | 482   |
| api_esp32.js      | 01-Jan-1970 00:00  | 460   |
| api_ota.js        | 01-Jan-1970 00:00  | 278   |
| conf9.json        | 01-Jan-1970 00:00  | 226   |
| platform.js       | 01-Jan-1970 00:00  | 154   |
| conf0.json        | 01-Jan-1970 00:00  | 3     |


The 1970 date obviously sticks out as the unix epoch time. There wasn't too much info in here that I could use. There is a reference to mqtt, rpc and uart, which revealed some things I may be able to use to get into the device, but it's mostly standard js.

Fun fact, the conf9.json contains your wifi SSID and password in plaintext. I would expect they would use some kind of encryption to be honest.

I tried to fuzz the directory

```
dirb http://10.0.0.145 common.txt -S
```

It only found one directory which returned HTTP invalid request:
http://10.0.0.145/rpc

Trying to connect via RPC didn't really work as I didn't know what commands to send yet.


# Reverse engineering the app
I decided to investigate the app instead. I used a rooted version of Android by using an Android Studio Android emulator. 
Getting everything to work was a bit of a balancing act — I needed a version of Android that both supported the Play Store (required for the Pit Boss app) and could also be rooted. Eventually, I landed on an emulated Pixel 6a running Android 13.0 (API 33), which ticked both boxes. I used [rootAVD](https://github.com/newbit1/rootAVD) to root the virtual machine.

I installed [PCAPdroid](https://play.google.com/store/apps/details?id=com.emanuelef.remote_capture) to run a packet capture. The app allows for TLS decryption, which I turned on, but the app stopped working. I realised that it must not be trusting the PCAPdroid certificate as it was installed as a user certificate. I needed to turn that user certificate into a system certificate to get around this issue.

After doing a bit of research, I found a plugin for Magisk called [Trust User Certs](https://github.com/NVISOsecurity/MagiskTrustUserCerts) which moves certificates from a user certificate:

![User Certificate](https://i.imgur.com/YXVlIy2.png)

To a system certificate:

![System Certificate](https://i.imgur.com/DQIJ4Lq.png)

Once that was done, I was able to run a packet capture and decrypt the 

