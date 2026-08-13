---
title: Hacking my BBQ
description: Smoking the stack for fun and for profit.
date: 2026-08-13T00:40:04+10:00
draft: false
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

Once we browse to its webpage, we see a bunch of files, which appear to be Javascript, certificate and json files that the smoker uses:

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

Fun fact, the conf9.json contains your wifi SSID and password in plaintext.

I tried to fuzz the directory
```
dirb http://10.0.0.145 common.txt -S
```

It only found one directory which returned HTTP invalid request:
```
http://10.0.0.145/rpc
```

Trying to connect via RPC didn't really work as I didn't know what commands to send yet.


# Reverse engineering the app
I decided to investigate the app instead. I used a rooted version of Android by using an Android Studio emulator.
Getting everything to work was a bit of a balancing act. I needed a version of Android that both supported the Play Store (required for the Pit Boss app) and could also be rooted. Eventually, I landed on an emulated Pixel 6a running Android 13.0 (API 33), which ticked both boxes. I used [rootAVD](https://github.com/newbit1/rootAVD) to root the virtual machine.

I installed [PCAPdroid](https://play.google.com/store/apps/details?id=com.emanuelef.remote_capture) to run a packet capture. The app allows for TLS decryption, which I turned on, but the app stopped working. I realised that it must not be trusting the PCAPdroid certificate as it was installed as a user certificate. I needed to turn that user certificate into a system certificate to get around this issue.

After doing a bit of research, I found a plugin for Magisk called [Trust User Certs](https://github.com/NVISOsecurity/MagiskTrustUserCerts) which moves certificates from a user certificate:

![User Certificate](https://i.imgur.com/YXVlIy2.png)

To a system certificate:

![System Certificate](https://i.imgur.com/DQIJ4Lq.png)

Once that was done, I was able to run a packet capture and decrypt the communications between the app and the Pitboss App's servers.

Finally I'm getting somewhere, I was able to use Ctrl + F to search for my username as a string, which led me directly to the specific request where I got the authentication URL:

![Wireshark](https://i.imgur.com/NxfGSdj.png)

From there, it was just a matter of turning on the packet capture, and capturing each specific feature of the app.

# Authentication
Authentication is done fairly simply, by using a HTTP POST with your login credentials for your pitboss account, eg:
```
curl -X POST https://api-prod.dansonscorp.com/api/v1/login/app \
  -H "Content-Type: application/json" \
  -d '{"email":"foo@bar.com","password":"PASSWORD"}'
```

This responds with:
```
{
  "status": "success",
  "message": null,
  "data": {
    "customer_cart": {
      "id": [ID],
      "shopify_customer_id": "gid://shopify/Customer/[CUSTOMER]",
      "cart_object": "[]",
      "created_at": "2024-12-30T02:12:17.000000Z",
      "updated_at": "2024-12-30T02:12:17.000000Z"
    },
    "token": "[token]",
    "token_expiration": "2025-07-09T10:35:28Z"
  },
  "errors": null
}
```
What's strange is that they seem to be using Shopify for this? I'm not certain why, as from my understanding shopify is for ecommerce, however I haven't used it in a development environment before so it may be normal. 

# Websockets
After digging a little further, I found that the app connects to the device using a websocket wss://socket.dansonscorp.com/to/PBL-XXXXXXXXXXXX, with the X's being the MAC address of the smoker. I used wscat to open a socket to the device, passing along the bearer token from the login response, and I found that it sends back a JSON with 2 strings.

```
wscat -c "wss://socket.dansonscorp.com/to/PBL-XXXXXXXXXXXX" \
  -H "Origin: https://socket.dansonscorp.com" \
  -H "Authorization: Bearer [TOKEN]"
```

As soon as the connection opens, the grill starts pushing status updates on its own:

```
Received: {"id":-1,"src":"PBL-XXXXXXXXXXXX","status":["FE0B010205090600090600090600090600090600010206020100000000000000000001000100000200000000FF","FE0C01020509060009060009060009060009060001020601020602FF"]}
```

I was a bit stumped, I was able to eventually figure out the encoding, and it turned out the answer had been sitting in plain sight the whole time. The `/api/v1/grills/{id}` response I'd already pulled contains a `control_board` object with `status_function` and `temperature_function` fields - literal strings of JavaScript that the app fetches from the API and evaluates locally to turn those hex blobs into readable numbers:

```js
if (!message.startsWith("FE0B")) {
  return null;
}
const parts = parseHexMessage(message);
const status = {
  p1Target: convertTemperature(parts, 2),
  p1Temp: convertTemperature(parts, 5),
  p2Temp: convertTemperature(parts, 8),
  smokerActTemp: convertTemperature(parts, 17),
  moduleIsOn: parts[24] === 1,
  fanState: parts[34] === 1,
  hotState: parts[35] === 1,
  motorState: parts[36] === 1,
  lightState: parts[37] === 1,
  primeState: parts[38] === 1,
  isFahrenheit: parts[39] === 1,
};
```

Shipping the parsing logic itself as a string of code from the backend is a wild design choice as it means Dansons can change the wire protocol per grill model without ever pushing an app update, at the cost of every client blindly evaluating whatever the server hands it. Once I had this, decoding the status strings was trivial - each hex pair maps to a probe temperature, a target temperature, or a boolean state like "is the auger motor running".

# Websocket commands
These are all the commands i've been able to glean. Sending commands turned out to be just as simple as receiving them. The socket accepts JSON-RPC style messages, and the one that matters most is `PB.SendMCUCommand`, which forwards a raw hex string straight to the smoker's control board:

```
{"id":8,"app_id":"h9rRx/S2vdA9iw==","method":"PB.SendMCUCommand","params":{"command":"FE0501000802FF"}}
```

The same `/api/v1/grills/{id}` response also had the full table of commands the control board understands, so I didn't have to guess at the hex:

| Name                           | Command      |
|---------------------------------|--------------|
| Get Grill Status                | `FE0B01FF`   |
| Get Grill/Probe Temperatures    | `FE0C01FF`   |
| Set Temperature to Celsius      | `FE0902FF`   |
| Set Temperature to Fahrenheit   | `FE0901FF`   |
| Turn Grill On                   | `FE0101FF`   |
| Turn Grill Off                  | `FE0102FF`   |
| Turn Light On                   | `FE0201FF`   |
| Turn Light Off                  | `FE0202FF`   |
| Turn Primer Motor On            | `FE0801FF`   |
| Turn Primer Motor Off           | `FE0800FF`   |

The 'Turn Grill On' command was not documented but inferred. I can confirm that it does work, but I **strongly** recommend you never turn the grill on remotely for safety reasons.

Setting a target temperature is a little more involved since it's built dynamically rather than a static hex string, but the logic for it was sitting right next to the table above:

```js
let _hundreds = Math.floor(arguments[0] / 100);
let _tens = Math.floor((arguments[0] % 100) / 10);
let _ones = Math.floor(arguments[0] % 10);
return 'FE0501' + formatHex(_hundreds) + formatHex(_tens) + formatHex(_ones) + 'FF';
```

So setting the grill to 220°F is just `FE0501` followed by the digits of 220 in hex, then `FF`. With this, I could turn the grill on, set a temperature, and watch the status messages roll in confirming it was heating up - all without touching the app.

From here on, the rest of this blog post won't be very structured, it will just be me Listing all the features of the app and interesting endpoints.

# Features
I've used the [undocumented API](https://github.com/apption-labs/meater-cloud-public-rest-api) for [Meater](https://meater.com/shop) before, which uses simple GET requests to request the temperature of the temperature probe, however these smokers seem to use websockets instead, which is probably a better idea than sending constant GET requests.

Beyond the websocket, there's a fairly standard REST API sitting behind `api-prod.dansonscorp.com` that the app uses for everything else - account, cart, notifications, and per-grill metadata:

```
/api/v1/profile
/api/v1/grills
/api/v1/grills/{id}
/api/v1/cart
/api/v1/notifications
/api/v1/customer-grills
/api/v1/marketings
/api/v1/mobile-home-screen
/api/v1/token/refresh
/api/v1/heroes
/api/v1/cook-data/{device_id}
/api/v2/firmware-platforms/{platform}?current={version}
```

`cook-data` was a nice find - give it a device ID and a start/end timestamp and it returns the full temperature history for a cook, which is presumably what powers the graphs in the app:

```
GET /api/v1/cook-data/PBL-XXXXXXXXXXXX?starts_at=2025-04-11%2008%3A53%3A44&ends_at=2025-04-11%2009%3A53%3A44
```

One genuinely odd thing I caught in the capture was a request out to `api-prod.dansonscorp.com/ddm/activity` with query params like `type=ytmusic` and `cat=youtu00c`, which looks suspiciously like a Google/DoubleClick ad-conversion tracking pixel. I'm not sure why a pellet grill app needs to be firing YouTube Music ad attribution events, but here we are.

# The takeaway
What started as "I wonder if I can turn my grill on from my phone without opening the terrible app" turned into a much longer rabbit hole than expected - packet capture, rooting an emulator, and reverse engineering an entire device control protocol out of a single JSON blob that wasn't really trying to hide anything. Turns out decoding the status messages took more effort than actually controlling the grill once you know the format. A good reminder that most "smart" devices are just a small embedded webserver wearing a trenchcoat.