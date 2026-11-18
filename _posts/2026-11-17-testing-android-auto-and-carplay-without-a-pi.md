---
layout: post
title: "Can a phone run Android Auto or CarPlay on a factory Honda radio?"
excerpt: "I tested HondaLink touch and vehicle data on Android and iPhone. The next step is getting Android Auto onto the Civic's HDMI input with just a phone."
date: 2026-11-17 17:10:00 -0500
published: true
comments: true
categories:
  - automotive
  - embedded-systems
tags:
  - honda
  - honda-civic
  - hondalink
  - hlauncher
  - android-auto
  - carplay
---

HLauncher already brings Android Auto and CarPlay to the factory touchscreen in my 2014 Civic using a Raspberry Pi. I wanted to see how much of that setup I could move onto a phone. The radio already takes video over HDMI and sends touchscreen input over Bluetooth through HondaLink. If the phone can handle both jobs, it could take the Pi's place. [How the current HLauncher setup works]({% post_url 2026-11-14-installing-hlauncher-on-a-factory-honda-radio %})

## What worked on Android and iPhone

**Both phones received touches from the factory touchscreen over HondaLink.** I also read vehicle data on both, including coolant temperature and parking brake status.

On Android, I left the Pi connected to HDMI and paired the radio with my Galaxy XCover Pro. Tapping the factory screen controlled the Android phone. The Pi supplied the picture throughout that test because I couldn't get usable wired video out of the Samsung.

On the iPhone, I connected a USB-C hub to the radio's HDMI input and put my test app on the factory screen at 720 × 480. The app showed where I touched, including two fingers at once. The picture sometimes glitched or went blank and then came back. I still need to isolate what caused the dropouts.

## Android Auto is the next test

I think Android Auto has the better shot at working on a single phone. Headunit Reloaded documents a Self Mode where Android Auto and its receiver run on the same Android device. The idea is to send that picture to the Civic over HDMI and pass the radio's taps and swipes from HondaLink into the Android Auto session. [Headunit Reloaded's Self Mode instructions](https://play.google.com/store/apps/details?id=gb.xxy.hr)

I still need to test the whole loop on one phone. The first milestone is a stable picture from the phone itself. Then I want to open Android Auto on the factory screen, use the radio touchscreen to navigate it, and see whether vehicle data keeps coming through during the session.

## Where CarPlay stands

The iPhone test proved that my app could draw to the radio and receive touches and vehicle data at the same time. I also got CarPlay to start over USB with Apple's CarPlay Simulator on the Mac. The remaining step is getting a CarPlay session running on the iPhone itself so my app can send it to the Civic. iCarCast documents a mode that runs CarPlay on the same iPhone, but I haven't reproduced it in my app. [iCarCast's “This iPhone” mode](https://github.com/rPlayAI/carcast-public/blob/main/docs/icarcast-user-guide.md#this-iphone)

## Have an Android phone I could test with?

If you have a spare Android phone with **wired video output over USB-C** and would be willing to lend or donate it for testing, please leave a comment or reach out. Let me know the exact model and whether you've already used it with a USB-C-to-HDMI adapter. A phone with known working HDMI output would let me test Android Auto and the Honda touchscreen together without the Pi.
