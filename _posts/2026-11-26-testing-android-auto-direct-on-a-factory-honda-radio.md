---
layout: post
title: "Testing Android Auto directly from an Android phone on a factory Honda radio"
excerpt: "Help test HondaLink Direct Lab: an experimental Open Headunit fork that sends Android Auto over HDMI and receives touch input through the factory HondaLink Bluetooth connection."
date: 2026-11-26
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
  - open-headunit
---

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe
    src="https://www.youtube.com/embed/j4NP3AVkAis"
    title="Testing HondaLink Direct Lab on a factory Honda radio"
    loading="lazy"
    referrerpolicy="strict-origin-when-cross-origin"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: 0;">
  </iframe>
</div>

[Watch the video on YouTube](https://www.youtube.com/watch?v=j4NP3AVkAis).

I've been working on another way to get Android Auto onto the factory Honda touchscreen: running the receiver on the Android phone itself, sending the picture and media audio over HDMI, and receiving the radio's touch input over Bluetooth.

The app is called **HondaLink Direct Lab**. It's a separate experiment from [my Raspberry Pi HLauncher setup]({% post_url 2026-11-14-installing-hlauncher-on-a-factory-honda-radio %}). This version doesn't need a Pi or a CarPlay dongle. You connect a compatible Android phone to the radio with a USB-C-to-HDMI adapter and an HDMI cable.

It's working on my **Samsung Galaxy S21 FE running Android 16 / One UI 8**, and I'm looking for people willing to test other setups. The interface is rough right now. I'm focusing on the connection, picture, touch, and audio before cleaning up how the app looks.

**This is an experimental test build, not a finished product. HDMI output support does not guarantee that your phone will work with this app or radio. Please read the compatibility notes and disclaimers before buying hardware or installing it. Feedback is required if you participate in testing, including when everything works.**

## What you need

- A factory Honda radio with the older **HondaLink Connect functionality and an HDMI input**. My reference car is a 2014 Civic. A HondaLink name or logo by itself doesn't establish compatibility; newer systems and other radio variants may work differently.
- An Android phone that supports **native wired video output over USB-C**, sometimes described as DisplayPort Alt Mode. A USB-C socket, fast charging, or wireless screen casting alone doesn't establish that support. Send me your full phone model before buying anything if you're unsure.
- A USB-C-to-HDMI adapter, such as [the adapter I've been testing](https://www.amazon.com/dp/B0DKTBFX7Q), plus an HDMI cable to the car's factory HDMI input. That listing is an example of my setup, not a guarantee for every phone or adapter revision.
- Google's **Android Auto** on the phone, with its developer headunit server enabled as described below.
- The **HondaLink Direct Lab APK** linked below.

This is a wired HDMI setup. Bluetooth carries the HondaLink connection and touch input; it doesn't replace the HDMI cable.

## Download the test build

[Download the current HondaLink Direct Lab APK from Google Drive](https://drive.google.com/file/d/1nZWIt2l_GmS8fdayPUqEd-fRycGSVHu4/view?usp=sharing).

[Download the Direct Lab 0.2 modified source code ZIP](https://drive.google.com/file/d/1wcOtwoZni0HMVhQnBBlJT2jyUvciqGK6/view?usp=sharing). This includes the source, build instructions, and license notices. You only need the APK to install the app. The source download is provided for anyone who wants to inspect, modify, or build it.

The APK is currently distributed through Google Drive, **not Google Play**. Only use the download linked from this post or a link I've sent you directly. Android may ask you to allow installation from your browser or file manager. Allow it for this installation, then turn that installation permission back off if you don't need it.

I'll update the build and these instructions as testing progresses. Please include the app version when reporting results; an older copy may behave differently. Future test updates may change setup steps or require you to configure the app again.

## First-time setup

Do all installation, setup, and testing while safely parked.

### 1. Install Direct Lab, then pair the phone with the car

Download and install the APK. You should see **HondaLink Direct Lab** in your app list.

**Before opening Direct Lab to connect, pair your phone through the car's Bluetooth phone-pairing menu.** Select the car's **HandsFreeLink** connection on your phone and confirm the pairing prompts on both screens. If it's already paired, make sure it's connected. Keep Bluetooth calling enabled for HandsFreeLink.

### 2. Start Android Auto's headunit server

1. Open your phone's **Settings** and search for **Android Auto**.
2. Open Android Auto settings, scroll down, and tap **Version** about ten times. Accept the developer-mode prompt.
3. Open the three-dot menu and choose **Start headunit server**.
4. Complete any first-run Android Auto permission or agreement prompts.

This is Android Auto's developer mode. **USB debugging, wireless debugging, ADB pairing, and rooting the phone aren't part of the normal setup instructions.** Some display problems may still need separate troubleshooting.

### 3. Connect HDMI and select mirroring on Samsung phones

Connect the adapter to the phone, and run the HDMI cable from the adapter to the car's HDMI input.

On the Samsung setup I've tested, go to **Settings → Connected devices → Samsung DeX → Connected display** and select **Screen mirroring**. Depending on the software version, the choices may be labeled **Extended** and **Mirrored**. Choose **Mirrored**.

The goal is for Direct Lab to render its own Android Auto picture on the external display while its connection screen stays on the phone. You shouldn't need to operate Android Auto in a floating DeX desktop window. Menu names and external-display behavior can differ on other phones.

### 4. Configure connected-display audio

In Samsung's DeX or connected-display settings, enable **Play sound on connected display**. If media still plays through the phone, **unplug and reconnect HDMI after changing that setting**. I had to reconnect it before the setting took effect on my phone.

The intended arrangement is **media audio over HDMI and phone calls over HandsFreeLink Bluetooth**. Direct Lab prefers HDMI for the Android Auto audio it decodes, but Android and Samsung still control system routing and audio from other apps. The app cannot guarantee or force every other app's output route.

### 5. Open Direct Lab, then HondaLink on the radio

1. With the phone paired to HandsFreeLink and HDMI connected, open **HondaLink Direct Lab**.
2. Allow **Nearby devices** and **Microphone** when prompted. Choose your paired radio if asked.
3. Open **HondaLink on the car's radio**.
4. Wait for the Android Auto connection and complete any remaining prompts on the phone.

After the initial setup, opening Direct Lab with HDMI connected should automatically start the connection using the saved radio. You still need to open HondaLink on the radio. After restarting the phone, you may need to start Android Auto's headunit server again.

The display session is designed to continue if you switch to another app on the phone. Please test that while parked. Use **Stop direct session** when you're finished.

## What I need you to test

Please send me feedback even if the setup works perfectly. Successful results help establish which combinations are usable.

- **Connection:** Does it connect on the first attempt? Does opening the app reconnect after stopping, unplugging HDMI, or restarting the phone?
- **Picture:** Is Android Auto landscape and filling the radio screen? Report black bars, stretched graphics, a DeX desktop, window controls, or display warnings.
- **Touch:** Try controls near every edge and corner, dragging, scrolling, and pinch to zoom in a supported map view. Tell me if taps are shifted or gestures fail.
- **Media:** Play music from Android Auto and check that it comes from the car speakers. Also note whether music started directly from the phone behaves differently.
- **Calls and voice:** While parked, test an incoming and outgoing call, microphone pickup, and steering-wheel controls for tracks, voice activation, and answering or ending calls. Tell me what each button actually does.
- **Background operation:** Switch to another phone app and check whether Android Auto, audio, and radio touch keep working.

**Call handling and steering-wheel voice activation are still areas I need help validating.** My test Android phone isn't my primary phone, so I haven't fully verified those paths. Don't assume calls, voice commands, or every steering-wheel button will work properly yet.

If you want to try the optional voice bridge under **Advanced settings and tests**, Direct Lab can offer a **Choose default assistant for steering-wheel voice** option. Selecting it changes the phone's default assistant and is intended to forward activation into a running Android Auto session. It isn't a general-purpose phone assistant, and Honda's steering-wheel behavior still needs testing. Restore your usual default assistant when you're done.

## How to report results

Reply in the comments, email me at [evan@evanhorsley.com](mailto:evan@evanhorsley.com), or reach out on Facebook if I've talked with you there with:

- Your full phone model, including the model number if available, and Android / One UI or other manufacturer software version.
- Your car's year, model, trim, and radio type.
- Your adapter model or product link, plus the Direct Lab version you installed.
- Which parts worked and which didn't, including the steps that reproduce a problem and the exact error message.
- For a failure, the file from **Share full diagnostic log** in Direct Lab. Send it soon after the problem, before clearing the app's data or cache.

The on-screen log only shows recent events, so a screenshot can miss the useful part. The exported file includes more connection history and display information. A photo of the radio is also useful for sizing or touch problems.

Review logs and photos before sharing them. They may reveal device names, connection details, or information visible on your screen. Don't post phone numbers, contacts, addresses, or other private information publicly; send diagnostic files privately if you're unsure. Tapping **Share full diagnostic log** opens your phone's share flow so you can choose where to send it.

## Known limits and compatibility

**Even a phone with working USB-C HDMI output isn't guaranteed to work here.** Android version, manufacturer display behavior, Android Auto updates, adapter, HDMI negotiation, audio policy, and the factory radio can all affect the result. A successful test on one phone doesn't certify every phone in that family.

Samsung can report a **720×480 physical display mode** while giving apps a **1600×900 logical canvas**. Those are different measurements, and seeing 720×480 in a log doesn't by itself prove the radio picture has the right proportions. Earlier testing required an ADB display-size adjustment. Direct rendering improves our control over the picture, but an ordinary app can't silently override every manufacturer display restriction. If you see bars or incorrect touch alignment, send the log and a radio photo rather than assuming the adapter is faulty.

The app also isn't a kiosk or a way to lock down Android. It can't guarantee that manufacturer system controls, the DeX trackpad, or other phone features will be unavailable. Background operation, audio routing, calls, reconnection, and voice controls all need testing on each setup.

I can't promise compatibility, a fix for every reported issue, or a timeline for support. Keep a working alternative for navigation and calls.

## Built on Open Headunit

HondaLink Direct Lab is a **modified fork of [Open Headunit](https://github.com/andreknieriem/open-headunit)**, maintained by André Rinas and contributors, with earlier work originating in [Michael Reid's headunit project](https://github.com/mikereidis/headunit). Open Headunit supplies the Android Auto receiver foundation. My changes add the HondaLink Bluetooth integration, direct external-display rendering, touch handling, and phone-side setup flow.

This isn't a new implementation of Google's Android Auto. Google's Android Auto still runs on the phone, and the fork provides the receiver side. You don't need a second copy of Open Headunit installed to use Direct Lab.

Open Headunit is licensed under the **[GNU Affero General Public License, version 3](https://github.com/andreknieriem/open-headunit/blob/main/LICENSE)**. The fork retains the applicable license and copyright notices; individual bundled components retain their own licenses. The modified source for the distributed build belongs alongside its APK download. Credit for the receiver foundation belongs to the upstream authors and contributors, and Direct Lab-specific bugs should be reported to me rather than treated as upstream Open Headunit bugs.

## Test-build and safety disclaimer

HondaLink Direct Lab is independent experimental software. It isn't affiliated with, sponsored by, endorsed by, or certified by Honda, Google, Samsung, Open Headunit's maintainers, or the adapter manufacturer. Product names and trademarks belong to their respective owners.

The software is provided **as is and as available, without warranties to the extent permitted by applicable law**. There is no guarantee of compatibility, availability, reliability, fitness for a particular purpose, continued support, or uninterrupted operation. Test builds may crash, disconnect, route audio incorrectly, lose settings, or stop working after phone or Android Auto updates. Download availability and future features may change.

Only participate if you're comfortable installing an experimental APK, granting the permissions needed for Bluetooth and microphone functionality, and reporting problems. Follow Android's security prompts; don't disable device-wide protections or grant unrelated permissions just to make a test build install. This post isn't a substitute for a separate privacy policy or the software's license terms. Google Drive, YouTube, Android Auto, and any navigation or media services you use have their own terms and privacy practices.

Set up, test, and troubleshoot only while safely parked. Secure the phone and cables away from pedals, airbags, and vehicle controls. As the driver, you're responsible for safe operation and following applicable laws. Don't watch video, adjust the app, or use distracting features while driving. Don't rely on this experimental setup for emergency calls or safety-critical functions, and keep an independent way to make calls and navigate.

To the extent permitted by law, the authors and contributors accept no liability for losses arising from installation, incompatibility, interrupted calls or navigation, misuse, distracted driving, accidents, injury, property damage, or other use of the software. Nothing here excludes rights or liabilities that applicable law does not allow to be excluded.
