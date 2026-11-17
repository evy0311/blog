---
layout: post
title: "Installing HLauncher on a factory Honda radio: CarPlay, Android Auto, and more"
excerpt: "What to buy, how to flash a Raspberry Pi, and how to set up HLauncher on your factory HondaLink touchscreen for CarPlay, Android Auto, and other plugins."
date: 2026-11-14
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
  - carplay
  - android-auto
  - raspberry-pi
---

Introducing [HLauncher](https://github.com/evy0311/HLauncher), a Raspberry Pi app launcher that brings Apple CarPlay and Android Auto to compatible factory Honda touchscreens. If you're like me, you know know how good looking the interior is in these cars. Replacing the factory headunit with an aftermarket one just isn' the vibe; it
tends to take away from the original, 2010s look of the car. That's why I built HLauncher.

On a compatible Honda, the factory HondaLink app opens HLauncher on the original touchscreen. The Pi sends picture and media audio over HDMI, and the radio sends touch input back over Bluetooth. The factory radio keeps its original firmware.

The project started with [getting CarPlay to work through HondaLink]({% post_url 2026-10-22-carplay-through-hondalink-on-the-civic %}). I built HLauncher around plugins so I could add features to the same screen. The live gauges show vehicle data through Honda's built-in protocol. Support for OBD adapters is coming soon. It's available as a downloadable image for the Raspberry Pi 4.

<figure>
  <img src="{{ '/assets/posts/2026-11-14-installing-hlauncher-on-a-factory-honda-radio/civic-carplay.jpg' | relative_url }}" alt="Apple CarPlay on the factory touchscreen of a 2014 Honda Civic">
  <figcaption>Apple CarPlay running through HLauncher on my Civic's factory radio.</figcaption>
</figure>

## Check your radio before you buy anything

Start with the radio and the ports in your car. Look for the older **HondaLink Connect** app on the touchscreen and a factory **HDMI input**. You need both for this setup.

<figure>
  <img src="{{ '/assets/posts/2026-11-14-installing-hlauncher-on-a-factory-honda-radio/civic-console-ports.avif' | relative_url }}" alt="Factory HDMI port, USB port, and 12 V accessory socket in the Civic's front console">
  <figcaption>The Civic's factory HDMI, front USB port, and 12 V accessory socket.</figcaption>
</figure>

My reference setup is the HDMI-equipped factory radio in my **2014 Civic**. Some **2013–2015 Civics** with the same radio are candidates, as are some **CR-V, Fit, and HR-V** trims with the older HDMI and HondaLink setup. I haven't confirmed the full year and trim list. Check the actual radio and ports in your car before ordering parts, even if your model is on that list.

## What to buy

The 2 GB Pi 4 I started with is still enough to get HLauncher running. If you've already got one, I'd use it. Here's where I'd put the money if I were buying parts today.

| Build | What I'd buy | When I'd choose it |
| --- | --- | --- |
| **Lowest cost** | Pi 4 Model B with **2 GB RAM**, a good **32 GB Class 10 microSD card**, a **protective Pi 4 case**, the cables below, and a **12 V USB-C car adapter**. Add the Carlinkit only if you want CarPlay. | You want to get it running with as few purchases as possible. Reuse a Pi 4, card, or suitable charger you already have. |
| **My everyday pick** | Same setup, with a **4 GB Pi 4**. | You're buying from scratch. The 4 GB board felt smoother in my testing. |
| **Upgrade later** | Keep the same Pi and card. Add a fan if your case has a mount and heat becomes a problem, and swap the car adapter for a power controller when mine is ready. | You want a tidier install, a clean shutdown, or faster reconnects after short stops. |

The **Pi 4 Model B with 2 GB is the minimum I'd recommend**. Any Pi 4 Model B with 2 GB or more is a reasonable choice; there's no need to replace one you already own. I wouldn't buy a Pi 5 for this. It's more power hungry and generally costs more, and the extra performance is overkill here. If you already have a Pi 5 and want to experiment, the image in this guide targets the Pi 4, so treat the Pi 5 as a separate, unverified setup. [Raspberry Pi lists the power requirements for each model here.](https://www.raspberrypi.com/documentation/computers/getting-started.html#power-supply)

Whichever tier you pick, you'll also need:

- **Micro-HDMI to full-size HDMI cable.** The small end goes into the Pi; the full-size end goes into the Honda. About 1 ft works if the Pi sits next to the port. Measure the cable route before buying if you're mounting the Pi farther away.
- **USB-C power cable and a suitable car adapter.** A 1 ft cable is enough if the Pi is next to the adapter. Choose a longer cable if your mounting location calls for it. Look for **5 V at 3 A on the port powering the Pi 4**. The headline wattage on a charger doesn't tell you what it supplies at 5 V. I've been using an Anker 12 V adapter; this [Anker 323 listing](https://www.amazon.com/dp/B0BPH6S4DN) is an example, and [Anker lists 5 V at 3 A on its USB-C output](https://www.anker.com/products/a2735). Any reliable in-car supply that meets the Pi's requirements is fine.
- **A way to connect the microSD card to your computer for flashing.** Use the built-in microSD slot if your computer has one. For a full-size SD slot, use a microSD-to-SD adapter. If you have neither, get a USB microSD card reader that fits your computer's USB-A or USB-C port.
- **A protective Pi 4 Model B case is required.** Choose one that covers the entire GPIO header with the lid installed. Loose metal objects can short exposed pins and damage the Pi. Look for ventilation, a fan mount, or both. A fan is optional, but I'd use one in a hot climate or add one later if you notice heat-related slowdowns. I also plan to offer 3D printed cases. [Example: iUniker Pi 4 case with a 40 mm fan](https://www.amazon.com/dp/B07D3S4KBK).

### Pick the right phone connection

**CarPlay (iPhone)** requires a **Carlinkit AutoKit USB dongle** designed to **add CarPlay to an aftermarket Android head unit**. Look for a listing that says it requires the **AutoKit app or APK**. The **CPC200-CCPA** is one example. Skip the adapters sold for cars that already have factory wired CarPlay. [Carlinkit listing](https://carlinkitcarplay.com/products/carlinkit-wirless-adapter-for-android-head-unit-support-carplay-and-android-auto?variant=42882540732586) · [Amazon listing](https://www.amazon.com/dp/B09ZQF9YS4)

In this setup, the dongle plugs into the **Raspberry Pi**. HLauncher talks to it directly, so there's no AutoKit app to install on the Honda radio. I'm exploring ways to run CarPlay without the dongle. For now, it's required.

**Android Auto (Android):** No dongle needed. Bring a **USB data cable** that fits your phone and a USB-A port on the Pi. HLauncher's Android Auto connection is wired, so a charge-only cable won't work.

## 1. Flash HLauncher to the microSD card

1. Go to [HLauncher Releases](https://github.com/evy0311/HLauncher/releases) and download the **Pi 4 disk image** named like `HLauncher-VERSION-pi4.img.xz`. Download the checksum from the same release and [verify the image before flashing](https://github.com/evy0311/HLauncher/blob/main/docs/install.md#download-and-verify). The GitHub source archive and the `.hlu` update file won't create a bootable card.
2. Insert the microSD card into your computer's card slot or connect it using your card reader. Open [Raspberry Pi Imager](https://www.raspberrypi.com/software/) on your computer. Choose **Raspberry Pi 4**, then **Use Custom** for the operating system and select the downloaded `.img.xz` file. You can also use [balenaEtcher](https://etcher.balena.io/) and choose **Flash from file**.
3. Select your microSD card. **Flashing erases the selected card**, so double-check the drive. If Imager offers OS customisation, skip it. Write the image, let verification finish, then eject the card.

The image bundles HLauncher and the phone projection software. If you prefer to set up an existing Raspberry Pi OS install yourself, follow the [source installation guide](https://github.com/evy0311/HLauncher/blob/main/docs/setup.md).

## 2. Connect the Pi and pair it with the radio

Do the setup while parked. Keep power steady through the first boot and any storage changes.

1. With the Pi off, insert the flashed card. Connect the Pi's **HDMI0** port (the micro-HDMI port closest to USB-C power) to the car's HDMI input.
2. For CarPlay, plug the Carlinkit into one of the Pi's USB-A ports. For Android Auto, you can plug your phone directly into a Pi USB-A port now or after the wizard. The Pi has no USB data cable to the Honda radio.
3. Turn on the radio and power the Pi from your car adapter. Within about five minutes of the Pi booting, open **Settings → Phone → Connect a phone → Add Bluetooth device** on the Honda. Select **HLauncher** and confirm pairing.
4. Open **HondaLink** on the radio to start the HLauncher wizard. Choose the apps you want, pick your startup screen and theme, and set up Wi-Fi if you want it. If you're using the 12 V adapter, enable **Storage protection**, wait for it to finish, and restart when prompted.

## 3. Connect your phone

- **CarPlay:** Open the **CarPlay** tile in HLauncher. For wired CarPlay, connect your iPhone to the **Carlinkit adapter's USB socket** with a data cable. For wireless CarPlay, follow the adapter's pairing prompts on your iPhone. Accept the CarPlay prompt on your phone.
- **Android Auto:** Open the **Android Auto** tile and connect your phone **directly to the Pi** using the USB data cable. Unlock the phone and approve the first-run Android Auto prompts.

HLauncher is in active development. Check the notes for the release you download before relying on calls or voice controls. I'll keep adding features and improving compatibility for both CarPlay and Android Auto. If you find a bug, please include your HLauncher version, your setup, and what happened. [Report a bug on GitHub](https://github.com/evy0311/HLauncher/issues)

If the radio can't find **HLauncher**, check that the Pi has power and try within the first five minutes of boot. The [first-boot and recovery guide](https://github.com/evy0311/HLauncher/blob/main/docs/onboarding.md) explains how to reopen pairing. If Bluetooth pairs but the screen stays blank, check that the cable is in **HDMI0** on the Pi and that you've opened **HondaLink** on the radio. If Android Auto doesn't see your phone, try a known-good data cable connected directly to the Pi.

## Where I mounted the Pi in my Civic

The storage cubby was the easiest place to start in my Civic. It's close enough to the factory HDMI port to use a short cable, and you can plug straight into the port as it sits in the trim. I wanted the cable out of sight, so I unclipped the port from the trim and plugged in behind the dash. The port is still connected to the radio.

I also unplugged the cable for the factory front USB port from the back of the radio and used an adapter like the iDatalink ACC-USB2 to connect it to the Pi. That lets me use the original port as a USB connection to the Pi. For wired CarPlay, I plug the Carlinkit into that port and connect my iPhone to the Carlinkit's USB socket with a data cable. For Android Auto, I unplug the Carlinkit and connect my Android phone directly to the front USB port with a data cable. That port can't be used for Android Auto while the Carlinkit is plugged into it. Reusing the factory port is optional. You can make the same connections at the Pi itself. [iDatalink ACC-USB2 at Crutchfield](https://www.crutchfield.com/p_794ACCUSB2/iDatalink-USB2-USB-Port-Adapter.html)

I'm looking at a more permanent spot near the fuse box. From there, I'd run the HDMI cable and the phone's USB data cable behind the dash and along the footwell toward the center of the cabin. If you mount the Pi farther from the ports, measure the full route before buying cables. You may need a longer HDMI cable, USB data cable, or power cable. Keep any wiring in the footwell clear of the pedals.

That's how the ports are laid out in my Civic. The CR-V and other Hondas can put the HDMI and USB ports in different places, so check your own car before choosing a mount or ordering cables. [Honda's 2015 CR-V port locations](https://owners.honda.com/utility/download?path=%2Fstatic%2Fpdfs%2F2015%2FCR-V%2F2015_CR-V_Audio_Operation.pdf)

## Powering the Pi from the 12 V socket

I'm still using the 12 V USB adapter in my Civic. It's a good place to start, and there's one behavior you'll notice when you turn the car off. The factory 12 V accessory socket in these Hondas switches off when you turn the car fully off, so the adapter cuts power to the Pi immediately. Enable **Storage protection** in HLauncher to reduce the chance of filesystem damage from the abrupt shutdown.

I'm working on a plug-and-play power controller that will give the Pi time to shut down cleanly and let you choose to keep it up for a while after turning the car off. The quick reconnect after a short stop comes from the Pi already being booted. You can install HLauncher with the 12 V adapter today and swap the power setup later.

I plan to make a video showing HLauncher running in my Civic and a detailed installation tutorial that walks through the full setup.

## Setup help and compatibility feedback

If you have questions about your setup or get stuck during the install, leave a comment below. If you already have my email or know where to reach me on social media, feel free to message me there. I'm happy to help.

If you'd like me to handle the prep, I can help you pick out each part, order the hardware, and flash HLauncher onto the microSD card for a small setup fee. Message me directly by email or on social media with your Honda's year, model, and trim, and let me know whether you want CarPlay or Android Auto. We'll go over the parts cost and my setup fee before I order anything.

If you try this on another Honda, leave a comment with your **year, model, trim, and which radio you have**, even if it doesn't work. That'll help turn the candidate list into something people can use before they buy parts.

## HLauncher disclaimer

HLauncher is an independent community project with no affiliation, sponsorship, or endorsement from Honda, Apple, Google, or Carlinkit. It's provided as is, without warranty. Use it at your own risk, verify compatibility with your setup, and keep it separate from any safety-critical vehicle functions.

As the driver, you're responsible for operating the vehicle safely, keeping your attention on the road, and following applicable traffic laws. Set up, test, and adjust HLauncher while safely parked. While driving, refrain from operating HLauncher, watching video, or using features that distract you. Passengers must avoid distracting the driver.

To the extent allowed by law, the authors and contributors accept no liability for distracted driving, misuse, accidents, injury, property damage, or other losses arising from use of the software. Any rights or liabilities that the law protects still apply. [HLauncher's full disclaimer](https://github.com/evy0311/HLauncher#disclaimer)
