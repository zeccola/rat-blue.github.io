---
title: Advanced Communications
description: Ever thought about how to communicate in a post apocalyptic world where all known communication methods are down?
date: 2024-04-22T00:40:04+10:00
draft: false
tags: [sdr, radio, baofeng, rtl-sdr, survival, communication, antennas, lora, morse-code, diy, off-grid, ham-radio, mesh-networking]
---

So I've thought about this quite a few times - Imagine a situation in which there is a nuclear war or apocalypse, and society as we know it has fallen apart. There are gangs / clans roaming and controlling certain parts of the streets - how do you communicate with fellow people or groups without the internet?


# Existing Infrastructure

One method would involve utilizing existing infrastructure. If you've ever explored the [maps](https://ergon.maps.arcgis.com/apps/webappviewer/index.html?id=5a53f6f37db84158930f9909e4d30286) of communication / power lines, you'll see they branch almost everywhere. Utilizing existing infrastructure is probably the best bet to get reliable, long distance communications.  

While reusing Fiber Optic or Coaxial cables may not be feasible, using unused power lines or unused telephone cables could be an effective way to communicate over long distances.

Existing technologies do exist for this application, called [Broadband over power lines](https://en.wikipedia.org/wiki/Broadband_over_power_lines) and can reach up to gigabit speeds. This is of course assuming you can get your hands on such esoteric hardware in an apocalyptic situation.

You could use a super simple setup of having Person A with a power source on 1 end of the lines, and Person B connect an LED up to the other end of the line. Person A would then pulse the power on and off for super simple morse code communication. Of course resistance would be an issue here, but with enough voltage it is possible.



# Good old fashioned Radio

I love radios. You can buy some pretty good radios for under 50 dollars, such as the [Baofeng UV5R](https://baofengtech.com/product/uv-5r/), to achieve ~10km of range. They are a cheap, efficient and effective way to communicate over a long distance, 

You can make a [repeater](https://www.instructables.com/Baofeng-UV5R-Repeater/) with 2 of these radios, by connecting the input of one to the output of another. If you place this in a high spot, it can effectively increase the range, as the repeater will transmit back anything it receives. A strategic use of this would be to place it high up on a hill or other vantage point so that you could communicate around obstacles, or simply increase your range.

![Map of Melbourne's train lines](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f1/Repeater-schema.svg/1200px-Repeater-schema.svg.png)



# Big antenna

At the end of the day, antennas are just big hunks of metal used to pick up radio signals. Yes, they are tuned for specific frequency ranges but in simplistic terms, anything metal can pick up radio signals.

I'm not entirely sure if it would work but it is technically possible if you had a handheld radio and a cable to attach it to a structure. I've had experience doing this exact thing with a [RTL SDR](https://www.rtl-sdr.com/rtl-sdr-blog-v4-dongle-initial-release/), and I was able to pickup some FM radio channels by connecting to my metal coat rack.

I also used to be able to pickup AM radio stations when I used to pump the gain up on my guitar amp, the guitar pickups would pickup the AM radio via the guitar pickups.

I've seen it [demonstrated](https://www.youtube.com/watch?v=UNVW2vAaaQs) before, in fact, this specific video gave me the idea for this article. I've always wondered if it's possible to use larger metal structures for a radio antenna, a couple ideas could be:

1. Steel bridge
2. Metal cladding / railing 
3. Railway tracks
4. Light poles 
5. Tin Roof? Not sure if this one is possible

Just about any large object of metal is possible. In the video linked above, the youtuber VK3YE was able to transmit / receive on the 7-28mhz band fairly reasonably by connecting to a 5m sign pole.


# Railway tracks

This one is similar to method 1, but utilises train lines. Train lines, especially in my home city of Melbourne, radiate around the cit, with lines branching off the main city loop. It would be possible to communicate with anyone along the same line.

![Map of Melbourne's train lines](https://i.imgur.com/KzzbxnY.jpeg)

Train lines technically have a [few volts across them](https://en.wikipedia.org/wiki/Track_circuit) under normal operation to test if a train is currently travelling along a section of the track. A super simple solution is to connect a power source to the rails (one for positive one for negative) and an LED to the other end, and use it to pulse morse code.

# Laser communications

This is an interesting one, it's not too stealthy but could be used in a pinch. 

It would likely require line of site, unless you shine the laser up into the air and use it like a beacon. Again, morse code could be used here, or a similar such system to communicate. Maybe different groups could use different colours? You can even use Infrared or Ultraviolet lasers to be more stealthy as these are harder to see with the naked eye.

In some conditions, the beam of a laser can be seen up to [25km away](https://www.youtube.com/watch?v=FSf8dqB5LX4). Lasers over 1mw are restricted in Australia, but you can easily [source one](https://www.instructables.com/Powerful-Burning-Laser/) from a DVD-RW player.

Again, morse code could be used here, or you could make some fancy fixed communication system with a detector on one end.


# LoRaWan

[LoRaWan](https://lora-alliance.org/about-lorawan/) is an interesting idea, it's a [cheap](https://store.rakwireless.com/collections/lorawan-modules) low power wide area network protocol wherein you can share wireless connections over long ranges to IoT devices, with a range up 10km. Ideally you would set these up to be similar to radio repeaters, dotting them along every 10km so you can communicate with anybody along the line, where each LoRaWan

You could also set them up as a mesh as well, similar to the image below, and use them as a quasi mobile network.

![LoRaWan mesh configuration](https://brentonpoke.github.io/lorablog/Research%20in%20LoRa%20The%20Disaster%20Communication%20Networ%20f2ace93789e0439ea7d37ae301d1dfed/image3.jpg)

I might do another post about the system I envision, but you could ideally set this up for basic text or audio communications.

# Thermal Imaging

This isn't the most efficient, but still a possible type of communication. It would emit no radio, sound or light waves as well, making it quite silent, even if it's inefficient. The setup would be to have 2 people with line of sight of one another where they need to communicate silently. 

They could each hold up 1 or 2 warm objects, with 1 signifying distress for example, and the other party would point their thermal camera to view what is being communicated.

It's a long shot, but an interesting form of communication.


# Kite antenna

This one builds on the idea of improving radio antennas. Distance for radios significantly drops off when there are obstructions in the way of the transmitter and receiver. Increasing the antenna height could mean you get significantly better range and sound quality by attaching the antenna to a kite.

I've [seen it done](https://www.youtube.com/watch?v=1KAcf67Gkmc) with some pretty impressive results before.

