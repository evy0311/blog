---
layout: post
title:  "Inside the Honda Civic Radio"
excerpt: "A hands-on teardown of the 2014 Honda Civic radio, documenting the internal PCBs, processor, flash memory, RAM, and display hardware while laying the groundwork for deeper reverse engineering of the software and SD card."
comments: true
date:   2025-02-13 20:00:00 -0500
categories:
  - reverse-engineering
  - automotive
  - embedded-systems
---

<!-- Content -->

Today I was able to begin the first step in reverse engineering the radio in my 2014 Honda Civic Si. If you missed the original
post, go ahead and look [here](https://evanhorsley.com/blog/2025/02/12/new-car-and-reverse-engineering-it/) for more information.

The first step was a complete teardown to view the inner workings of the radio and see what is powering this thing.
I was also looking for some sort of serial or debug port, but more on that later. Enough chatter, lets look at the inside of this thing!

### The top layer PCB
#### Top
![Top layer of the radio showing various connectors, SD card slot, and fan header](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%201.jpeg)

The first picture here is the first/top layer of the radio. We can see several things at first glance. In the bottom left, there is a 6 pin wire that
goes down to the lower board. There are also two ribbon cables that connect the upper and lower boards. At the bottom middle we can see the display
connector. This connects in when the display is mounted in, similar to how a PCI port "clicks" in but there is no lock. We can also see an
SD card slot on the left side. In the very middle of the board is an empty connector that the CD drive plugs into via a ribbon cable. The top left
two pin header is for a fan.

#### Bottom
![Underside of upper section showing SoC, NAND flash chips, RAM, and audio processing chips](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%2019.jpeg)

This is the underside of the upper section. This is the real brains of the radio here, with a SoC living in the middle of the board. We
can also see two NAND flash chips, a RAM module, and a couple chips related to audio processing and power handling. Other than that, quite
boring.

### The bottom layer PCB
#### Top
![Lower section of radio showing audio processing hardware and connectors](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%206.jpeg)

This is the lower section of the radio. We can again see the 6 pin connector that goes to the upper board, as well as both bottom ribbon cables. There is also quite a bit
of audio processing hardware here mostly in the top left of the image. There are two main chips we can see, but more on those later. There
is also an empty ribbon cable connector towards the bottom right in beige that I cannot identity its use. It was unused when I opened it up.

#### Bottom
![Underside of lower section](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%2011.jpeg)

This is the underside of the lower section. Not too much going on here, really.

### The processor
![Renesas R8A77780 processor](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%2024.jpeg)

The processor appears to be a Renesas R8A77780. I unfortunately cannot find much about this online, but from what I can gather, it is a VERY low
power chip. 


### The ROM
![Macronix MX29GL640ELT2I-70G flash memory chip](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%2026.jpeg)

The flash memory appears to be a Macronix MX29GL640ELT2I-70G memory chip. It's a NOR flash memory device from Macronix's GL640E family with the following key specifications:
- 64 Megabit (8MB) capacity
- 70ns access time (indicated by the -70 in the part number)
- Temperature rated for industrial use (indicated by the I in the part number)

### The RAM
![Micron D9PTJ ROM chip](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%2028.jpeg)

The system RAM appears to be made by Micron, model D9PTJ. I can't find much online about this though, but will keep digging.

### The display
![Display assembly with Alps UGKZ2-E06A WiFi/Bluetooth chip](https://github.com/evy0311/blog/raw/gh-pages/assets/images/2025-02-13-inside-the-honda-civic-radio/teardown-pictures/honda-civic-radio-inside%20-%2029.jpeg)

The display piece is one assembly with all of the connections for digitizer, screen, and front buttons/lights all connecting to one PCB. That PCB connects,
like described earlier, directly to the internal PCB with a push in connector that lacks a lock. It just pushes in when the entire unit is assembled. Nothing too crazy
going on here. It is worth calling out the Alps combo WiFi/Bluetooth chip though, model UGKZ2-E06A. The WiFi aspect is important as it may be an attack vector that
could be exploited, but not sure.

## What Next
Now that we know what makes this thing tick, we may be able to have a better understanding of the software. Unfortunately, I did not see an obvious JTAG, Serial, or other debug interface/port. That doesnt mean that
there isn't one, though. My next steps are to research this hardware more, and figure out the exact function (and hopefully contents) of the SD Card. Interestingly, without it, the unit will not boot and displays "Preparing for update"
on the boot screen, then freezes with error "000006" or something similar. I also want to keep digging around the software, and map the hidden factory menus.

There are a lot more close up photos of the internals in full uncompressed format in a .zip file down below. Feel free to download and take a closer look. 

Thanks for reading, and stay tuned for more hopefully soon.

.zip file with full-size pictures: [honda-civic-radio-inside-full.zip](https://drive.google.com/file/d/1hphRFdYSxYR2MdTMsTRx1IH1QSYL00zD/view?usp=sharing)