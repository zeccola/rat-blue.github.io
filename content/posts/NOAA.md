---
title: RTL-SDR and NOAA Satellites
description: Running your own weather station is cool and all, but having a weather satellite station is even better.
date: 2025-04-07T00:40:04+10:00
draft: false
tags: [sdr, rtl-sdr, noaa, satellite, diy, radio]
---

I've had an [RTL-SDR V3](https://www.rtl-sdr.com/rtl-sdr-blog-v-3-dongles-user-guide/) kicking around for quite a while at home. They are a good piece of kit - essentially a USB radio receiver that receives from 500khz to 1.766ghz (and also has BIAS T support). I've used it for some cool 433mhz projects, as well as just listening to FM radio when I feel like it, but I had the idea to use it to read NOAA weather satellites.


# Weather Satellites
There are essentially 2 different types of weather satellites that are used (there are more, but they mostly follow these two types of orbit).

1. Geostationary orbits. These normally orbit at around 35,000km above the Earth's equator, they appear to be floating above us but in reality they orbit at the same speed as earth's rotation, so it's constantly looking at the same side of the earth. They typically provide good monitoring of specific regions which can make them useful for tracking hurricanes. [Himawari 8](https://en.wikipedia.org/wiki/Himawari_8) is a good example of this, it was launched by Japan in 2014. The issue with these is that they are very far away, and as such the signal from space is very weak, meaning it will be quite hard to pull images off it.

2. Sun-synchronous orbits. These are orbits that rotate around the earth, rotating around each place roughly at the [same time each day](https://en.wikipedia.org/wiki/Sun-synchronous_orbit), meaning that the light level will be fairly consistent given a pass over an area. These only orbit around 800km above earth, and as such can easily be connected to.

I've chosen the [NOAA 15 satellite](https://en.wikipedia.org/wiki/NOAA-15), as it passes over my city quite regularly at around 7am and 9pm.


# The setup
The setup was quite simple. I had a milk crate which had a spare Thinkpad T480s running Ubuntu 22.04 with [SDR++](https://www.sdrpp.org/), [N2YO](https://www.n2yo.com/passes/?s=25338&a=1) for tracking the satellite overhead, and [Satdump](https://www.satdump.org/) for converting the signal to a usable image. 

As for the SDR, I was using my RTL-SDR v3 with a DIY dipole made of cardboard and copper wire angled at 120 degrees. Dipoles work because each 'leg' of the dipole handles one half of the alternating radio wave. One leg carries the positive half cycle while the other carries the negative half cycle:

![Dipole antenna](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/Dipole_receiving_antenna_animation_6_300ms.gif/330px-Dipole_receiving_antenna_animation_6_300ms.gif)

Angled dipoles are simple to make, but they are directional so I used an app called [Stellarium](https://play.google.com/store/apps/details?id=com.noctuasoftware.stellarium_free&hl=en_AU) to track the satellite as it passed through the sky.

Technically, the best type of antenna for this setup is a [QFH antenna](https://wiki.rev0.net/Quadrifilar_Helix), but they are quite expensive, and DIY QFH's can be a pain to make. They're better than a dipole because they are omnidirectional, meaning you can just set it up in a single spot and never have to aim / move it.


# First attempt
Heres a lesson to be learned when it comes to preparation and thinking on the spot. I planned out a satellite pass on n2yo and found it would pass over at around 9pm, so I headed down to the beach with my laptop, the SDR and the antenna. I followed the signal as it went along the sky, but I wasn't seeing a signal on SDR++ - for some reason.

Not wanting to waste the pass, I pulled out my trusty [UV-5R](https://baofengtech.com/product/uv-5r/) and MacGyvered a solution by dialing into the frequency and recorded the speaker output by holding my phone against the speaker of the radio. To my surprise, it somewhat worked:

![First Attempt](https://i.imgur.com/fQXpUL8.png)

I'll admit, it doesn't look impressive at all, but at least you can still make out *something* resembling a proper weather picture. It was only once I was home that I realised the gain on SDR++ was set to practically zero! This explained why I couldn't pick up the NOAA satellite signal.


# Second attempt
For the second attempt, I made sure to have the gain turned up, and even used a [Low Noise Amplifier](https://en.wikipedia.org/wiki/Low-noise_amplifier) to make sure I could pick up the signal. This time, it was a huge success. Despite the pass being objectively worse than the original (Attempt 1 was practically directly overhead, this one was at a 50 degree angle from the horizon) I got a significantly better picture:

![Second Attempt](https://i.imgur.com/88qgolU.jpeg)

I was super happy with this pass, the image is super crisp and clean. You can even see down the bottom where the signal gets really poor due to it dipping below the horizon.


# Third attempt

For the third and final pass, I wanted to try something slightly different. I had made the beach trip twice now, so I wanted to see if I could pick it up from my house. I did this by sticking each leg of the dipole up against the window that faced the rough direction the satellite would be in. 

![Third Attempt](https://i.imgur.com/6cbL1RS.png) 

The image wasn't great. This is probably why Dipoles are typically aimed at the source, you don't get a great signal otherwise.


# Conclusion

Overall, I was really happy with this project. I had the idea of automating this to an [ESP32](https://en.wikipedia.org/wiki/ESP32) connected to an e-ink display, which would automatically update with the latest NOAA picture. I might still give this a try, but I would really need one of the QFH antennas that I was talking about earlier in this post.

Apart from that, moving forward I really want to try receiving a [HRPT](https://en.wikipedia.org/wiki/High-resolution_picture_transmission) signal in the near future. They're significantly higher resolution, but also higher frequency than the normal NOAA signals, meaning it's going to pull a bit of work to do correctly.
