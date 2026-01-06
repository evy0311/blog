---
layout: post
title:  "Attempting to get CarPlay on the stock headunit"
excerpt: "An experiment to bring CarPlay to a stock Honda head unit using its hidden HDMI input—leveraging a Raspberry Pi, reverse-engineering Honda’s phone projection system, and exploring touch, audio, and control limitations along the way."
comments: true
date:   2025-10-04 12:00:00 -0500
categories:
  - reverse-engineering
  - automotive
  - embedded-systems
---

<!-- Content -->

Well, for almost a year I had the Joying android headunit (Option 3 mentioned in my post [here](https://evanhorsley.com/blog/2025/02/12/new-car-and-reverse-engineering-it/)) in my 2014 Honda Civic Si. This was one of the
options I had mentioned in my original post a bit back. It was great, but I really missed the look of the stock
Honda headunit.  Something about it is just "better" and it looks great. The carbon fiber trim, the sleek black radio screen,
and the genuine Honda interface all just looked too good. I decided to switch back to the stock one, purely to give it a try.

The Joying unit is overall good. The wireless CarPlay worked great and I had 0 issues with that. My only complaint is the audio
quality. Its not the best, but not terrible either. But the stock headunit produces noticeably better audio, in my opinion. This was another reason why I switched back.

### The new experiment

After swapping back to stock, I had a goal: how can I get CarPlay on the stock Honda headunit? The system as we know runs
Windows CE, but getting into that interface has so far proven to be difficult, and even if we could, development for Windows CE is hard,
mostly given its age and deprecation. That leaves us with one main route: Utilizing the HDMI input.

My thinking is this: The car has an HDMI input, so why can't I just run a Raspberry Pi in the car that outputs CarPlay (or anything else I want, really) to
the car headunit? This is the least intrusive option, because then all the existing car function like backup camera, lanewatch, and audio control
remains untouched, and those things will still override when needed.

### The problem with this

Well, there are a few. First off, the HDMI input can't be used when driving. This should be easy to solve, because the headunit has an input wire on the back
that connects to the parking brake signal. If this input is grounded, the headunit will think the parking brake is on, and then the HDMI will work while driving. This can 
be achieved by either splicing into that wire and grounding it, or doing what I plan to do, and making a harness to sit in between the headunit and existing
plug to "intercept" this signal and ground it myself. Easy enough.

The second problem? How to control the Raspberry Pi when it's in the car. Since HDMI doesnt transmit touch data, we need a way to either get the touch data from the car
back to the Raspberry Pi, or an alternative method. Of course, touch screen is preferred, but also the most difficult. The easiest option is to wire in a rotary encode with push button
to the Pi, then that can be used to control CarPlay (similar to how some BMW and Mazda cars do). The rotary encoder option will be the first path I take, to get this going for the time being. 
However, I have a plan to get the touch data out of the stock unit and into the Pi.

Stepping back, the original purpose of the HDMI input was much less about plugging in devices like a PlayStation or Apple TV. It had everything to do with Honda's early attempt at a phone driven
infotainment system, pre-CarPlay and Android Auto. The idea was an iPhone could be plugged into the HDMI and USB ports on the car, and also connected via Bluetooth. Then, you would load the Honda
specific apps onto the phone, and then when open those apps could project things onto the car screen, even while in motion. For example, Honda had a navigation app that would display a GPS-like
interface on the car radio, and you could even interact with it via the touchscreen.

How this all worked together, I've no 100% understanding, yet. But my initial thoughts have been that the HDMI served the video, and possibly audio from the phone. Then, the USB was used to keep the phone charged,
and possibly served the touch screen data to the phone (since user's could interact with the Honda radio touchscreen and control the GPS apps running on the phone). The Bluetooth likely served for calls and to allow
steering wheel controls. 

I've done initial attempts to mimic this interface with no luck, yet. I think the car expects a very specific sequence of setup and commands from the phone over USB, of which I'm not sure what they are. The apps
are no longer available for download, and Honda has long discontinued them everywhere. If anyone has an old iPhone lying around with these apps, let me know, and I will buy it off of you. Ha. 

As for audio, I was initially hoping that the car would allow me to have the HDMI plugged in for video, but audio going over Bluetooth from the Pi. This would've made it much easier to relay the steering wheel controls
to the Pi and then to CarPlay, but the headunit views these as two separate inputs, so it wasn't possible from what I can tell. Audio over HDMI works fine, but of course, no phone button controls.

### What works so far

So far, I've gotten the Raspberry Pi displaying the CarPlay interface to the screen, and audio works great. I had to adjust the headunit's color settings for the HDMI input as it looked a bit washed out, 
but other than that I think it turned out pretty great! Now that this works and looks good, it's time to solve for all of the other problems. Those will be the hardest part. but I've gotten a pretty good understanding of this
system so far and will continue to work on this. 

For those wondering, CarPlay is running on the Raspberry Pi with software found [here](https://github.com/rhysmorgan134/react-carplay).

![CarPlay Sneak Peak](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-10-04-attempting-to-get-carplay-on-the-stock-headunit/sneak-peak.jpeg)
