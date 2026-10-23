---
layout: post
title: "CarPlay through HondaLink on the Civic!"
excerpt: "The factory HondaLink app is talking to a Raspberry Pi, displaying CarPlay over HDMI, and sending touchscreen input back over Bluetooth. Here's where things stand."
date: 2026-10-22
published: true
comments: true
categories:
  - reverse-engineering
  - automotive
  - embedded-systems
tags:
  - honda
  - honda-civic
  - hondalink
  - carplay
  - raspberry-pi
  - reverse-engineering
---

Well, that escalated quickly. Yesterday we had a working radio again. Today, we have CarPlay running through the factory HondaLink app on my 2014 Civic headunit!

There is a Raspberry Pi and a Carlinkit dongle involved, so this isn't the CR-V firmware suddenly working on the Civic. But the radio displays it, and its touchscreen controls it. That was the goal.

### Getting HondaLink to talk

The original HondaLink setup used an iPhone, HDMI, and a Bluetooth control connection. With the old apps discontinued, I wanted to see if a Pi could take their place.

We started with MirrorLink and got a working display and touch connection, but moving video through that path on the Pi Zero 2 W was painfully slow. HDMI already looked smooth on this radio, so the next question was whether we could use HondaLink for control while sending video separately.

This meant tracing the radio firmware, comparing Bluetooth captures against what the code expected, and gradually implementing the missing conversation on the Pi. Connecting Bluetooth wasn't enough. The radio also wanted the right app information, launch responses, and ongoing session messages before it would actually show anything.

There were a lot of "required apps are not installed" and "unable to start the app" screens along the way. Eventually, HondaLink opened our Pi launcher and stayed open. Then we figured out the touchscreen messages and could actually press things. Huge!

### What works now

The current setup uses a Pi 4 with 2 GB of RAM, with the Carlinkit dongle plugged directly into it. Video goes to the radio over HDMI, and touch input comes back over Bluetooth. No desktop environment is needed on the Pi.

CarPlay is working with taps and swipes, and the current pipeline is configured for 60 fps. The Honda icon inside CarPlay returns to our Pi launcher. Earlier attempts to share the dongle over Wi-Fi were laggy and sometimes froze completely; the direct USB setup has been much better.

We also got microphone audio from the radio over Bluetooth. There was a catch: outside an active hands-free call state, the Pi received perfectly valid PCM containing digital silence. A local simulated call state opened the microphone, and the live level meter finally moved when I spoke. That test is isolated from CarPlay for now.

### What's left

Audio is still unfinished. Microphone capture works in the test page, but it isn't connected to CarPlay yet, and CarPlay sound still needs routing. Startup, reconnects, and handling failed sessions also need more testing before I'd call this something ready to install and forget about.

All of this is still on the bench. But we now have a working video path, touchscreen control, and a confirmed microphone path. That's a pretty ridiculous amount of progress from a radio that was boot looping yesterday.

I'll let the video show the rest:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe
    src="https://www.youtube.com/embed/j8QVPoiYK9U"
    title="CarPlay through HondaLink on the Civic"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;">
  </iframe>
</div>
