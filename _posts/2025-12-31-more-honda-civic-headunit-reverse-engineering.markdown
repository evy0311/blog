---
layout: post
title:  "More Honda Civic headunit reverse engineering"
comments: true
date:   2025-12-31 08:00:00 -0500
categories: honda civic reverse engineering hack hacking car auto automotive
---

<!-- Content -->

I have been slowly working through how to make custom modifications to the factory headunit in my 2014 Honda Civic Si. As many of you probably remember, these cars were advertised around 2014 and 2015 as candidates for future software updates from Honda. Those updates were supposed to add features like Apple CarPlay and Android Auto. Honda ultimately abandoned that plan, and those updates never came.

The end result is that owners of the Civic, CR-V, HR-V, and Fit across several model years were left with a great-looking 7 inch display that is, for all practical purposes, a “dumb” screen. A few years ago, a beta firmware leaked that included Apple CarPlay, but it only worked on the CR-V without navigation. That still left most owners completely out of luck.

Going into this project, I honestly did not know how locked down this system really was. Everything was theoretical at first. I spent a lot of time researching this specific headunit and kept coming up short. No one seemed to have gone very deep into it. That is when I stepped back and looked at the bigger picture.

The first key question was who actually manufactures this headunit. It turns out they are built by Mitsubishi Electric Co., often referred to as Melco. With that information, I started looking at other Melco-built automotive headunits. Sure enough, there are several very similar “cousin” units, including the MMCS and NR-xxx series, which even share a similar model number scheme to the Honda units. These systems are known to run Windows CE, and there is a decent amount of documentation online about modifying them, including things like changing splash screens and gaining limited system access.

### Attacking the SD card

My first potential attack vector was the SD card. On many MMCS systems, users were able to gain access through removable media, although often via CDs or internal hard drives. If I could understand how the Honda SD card worked, there was a chance I could take advantage of that.

Plugging the SD card into a normal computer yielded no mounted volume. That tells us a few things. Either the card is protected with an SD password, it uses a non-standard partition layout, or both. The easiest first step was to determine whether an SD-level password was being used.

To do this, I used a logic analyzer (LA2016) along with a dedicated SD sniffer board from SparkFun. The setup allowed me to electrically sit between the headunit and the SD card and observe all traffic on the SD bus during boot. Using sigrok and PulseView, I captured the full initialization sequence as the headunit powered on.

Very quickly, something interesting showed up. During the early SD command sequence, the headunit issues a CMD42 command, which is the SD protocol command used for locking and unlocking a card. In other words, there is absolutely a password involved. Continuing to monitor the traffic showed the card being successfully unlocked, mounted, and then read from as the system booted.

### Unlocking the card for real

Knowing the password exists is only half the battle. Normal PC SD card readers cannot unlock a card protected this way. The SD password mechanism operates at the SD protocol level, before any filesystem is exposed, and consumer hardware simply does not give you access to that layer.

To work around this, I used a Teensy 4.1 wired directly to the SD card signals via the sniffer board. In this setup, the Teensy acts as a very simple SD host controller. I wrote a small program in the Arduino environment that speaks raw SD commands over GPIO. This allowed me to send CMD42 manually, provide the password, and then issue additional commands to read card metadata and raw sectors.

At first, the password I extracted did not work. That sent me back to the capture data. I then came across an excellent blog post describing a similar reverse engineering process on a VW navigation system. The author explained that SD command bit alignment can be misleading when viewed in a logic analyzer, and that the password bytes may be shifted relative to byte boundaries.

With that in mind, I adjusted the bit window I was using to interpret the password data. After realigning the bits and retrying the unlock command, it worked immediately.

### The SD card password

Here is the 16 byte SD password used by this headunit, formatted as hexadecimal bytes:

```
95 D5 9D E5
86 FD BD 85
8D DD F6 76
5D 96 FE FF
```

Once the card was unlocked, I was able to read the card ID (CID) and perform a full, bit-for-bit dump of the entire SD card.

### What is on the card

This was a huge breakthrough. The SD card contains all of the UI image assets as well as the Windows CE operating system files themselves. In other words, the headunit appears to boot directly from the SD card.

That immediately raised two important questions.

The first question was whether the headunit validates the SD card’s CID during boot. If it did, that would prevent using cloned or replacement cards. To test this, I wrote the full image to a completely different SD card and inserted it into the headunit. It booted without any issues. This confirms that the system does not verify the card ID, only the contents.

The second question was whether I could modify individual files on the card and have those changes show up on the headunit.

### Modifying the splash screen

The first experiment was the splash screen. After manually searching through thousands of image files, I found the splash screen that matched my specific car. I replaced it with a custom image, making sure the file size, format, and encoding remained identical to avoid triggering any integrity checks.

When I inserted the card and powered on the headunit, I initially saw the original splash screen. That was disappointing, but only for a moment. After a few seconds, my custom image appeared.

This suggests there is a fallback splash screen stored somewhere in the headunit’s internal storage that is shown early in the boot process, followed by the SD card version once the system is fully up. More importantly, it strongly suggests that at least some files on the SD card are not protected by cryptographic signatures or checksums.

### Reusing the password on other headunits

The next experiment was to see whether this SD card password was unique to my specific headunit, or if Honda and Melco reused the same password across multiple vehicles. This distinction matters a lot. If the password were unique per unit, reproducing this work would require extracting a new password from each headunit. If it were shared, then anyone could unlock their card using the same basic hardware setup.

To test this, I purchased an SD card from a 2016 CR-V with navigation and attempted to unlock it using the same CMD42 password. It worked immediately.

This strongly suggests that the SD password is reused across at least multiple Honda headunits and model years. Practically speaking, that means others should be able to reproduce this process at home using a Teensy (or similar microcontroller) and an SD breakout or sniffer board, without needing access to the original vehicle the card came from.


### Where this goes next

That is where I will stop for now. The fact that the unit boots from an easily clonable SD card, does not verify the card ID, and accepts modified image assets opens up a lot of possibilities.

In a future post, I plan to document the hardware setup, the Teensy code, and the tooling in much more detail. I also plan to continue digging into the Windows CE environment itself.

The goal of this post was to share progress and hopefully get others in the community excited about what might be possible.

---

## Images

![CarPlay Sneak Peak](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-12-31-more-honda-civic-headunit-reverse-engineering/pulseview.png)

## Video

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe
    src="https://www.youtube.com/embed/aiE3k4v_0Ao"
    title="Honda headunit reverse engineering"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;">
  </iframe>
</div>