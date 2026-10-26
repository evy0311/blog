---
layout: post
title: "Dumping the Civic headunit's internal flash"
excerpt: "CarPlay works through a Pi, but can the Civic run it natively? We now have a complete dump of the radio's main NOR flash, and a much better picture of what actually makes this thing boot."
date: 2026-10-26
published: true
comments: true
categories:
  - reverse-engineering
  - automotive
  - embedded-systems
tags:
  - honda
  - honda-civic
  - carplay
  - windows-ce
  - firmware
  - reverse-engineering
  - nor-flash
---

We have CarPlay working through HondaLink with a Raspberry Pi. Naturally, that means it is time to make this unnecessarily complicated again.

The next question is whether the radio can run CarPlay itself. Honda released a CarPlay update for the CR-V headunit, which is very similar to the one in my 2014 Civic. That package contains `CarPlay.exe` and the supporting software. We already know that convincing the Civic to accept the update is possible. We also know that doing that does not mean it will boot afterward. Learned that one the fun way.

This time, the approach has been to understand the pieces and test them individually on cloned SD cards, with the working original and recovery images preserved.

### Getting the app onto the radio is the easy part

Since the last post, we have made working home-screen modifications, added a sixth tile, and built small USB updates that change individual assets. We can replace both stages of the boot logo, including the one stored internally. Custom programs can also run from the radio's UI.

That gives us a way to launch experiments, but transplanting the CR-V software has been much less cooperative. Depending on the combination, we have seen boot loops, a home screen with broken touch interaction, and an updater that eventually complains about reading a CD-ROM. Some launcher attempts did nothing visible; others went black and rebooted.

So far, we have **not** demonstrated native CarPlay working. Copying the EXE is only part of it. The process needs compatible libraries, inter-process communication, drivers, and whatever the rest of the radio expects when a new source starts. Swapping a shared library can also break the existing Civic applications that use it.

We needed better visibility into the hardware and its saved configuration.

### An app to dump the internal flash

The SD card holds a large amount of the Windows CE software, but it isn't the whole radio. There is also internal flash containing boot code and persistent data. Until now, most of our understanding of that came from tracing the functions that read and write it.

We built a diagnostic app, launched from a home-screen tile, to collect information and write it to a USB drive. Then we extended the experiment to read the main NOR flash.

The first attempts failed. One produced a partial output file before the radio rebooted. Other read paths returned errors without giving us actual flash data. A file existing on the USB drive clearly wasn't enough to call this a successful dump.

The working version added a bounded, read-only operation to the SD-resident flash driver. That let the read happen in the driver context, using its existing flash mapping. The app requested small chunks, read each twice, compared the results, and saved a structured journal to USB. A simple progress display made this considerably less mysterious than clicking an icon and waiting.

The result was a complete **8 MiB main NOR dump**: 2,048 blocks of 4 KiB, with both reads agreeing for every block. The capture took about two minutes, including writing the journal. We checked coverage, decoded the journal independently, and preserved the original capture and hashes.

No flash erase or programming commands were used for the dump. We have identified write and erase routines in the driver, but that is not the same as having a tested restore procedure.

### What is actually on it?

Quite a bit! The current map includes:

| Area | What we found |
|---|---|
| Boot code | Startup and recovery/loading code |
| Internal opening image | An exact match for the custom Si logo we installed |
| SD boot metadata | Component locations, lengths, checksums, and bank selection information |
| Update records | The saved update request, manifest, and result/status |
| System and model parameters | Version strings, model identification, and parameter profiles |
| Diagnostic areas | Saved log fragments and embedded diagnostic code |
| Persistent settings | Named records used by the HMI, audio, player, tuner, and communication software |

The boot metadata was particularly useful. We decoded the active record and checked its enabled entries against the SD baseline. The updater, main OS, and resource-image lengths and checksums matched. This makes the relationship between the internal boot records and the SD contents much more concrete.

There is also a good reminder here about update state: the last update's manifest can remain populated after completion. Finding `SwUpdate.mef` and version strings in flash does not automatically mean the radio is still trying to update.

The settings storage has its own structure, with `BKUP03` headers, allocation tables, and checksums. We reconstructed 25 named records across 658 stored copies. Those include HMI settings, player resume information, tuner data, and communication settings. We haven't established which copy the manager selects as current in every case, or what every field means, so those copies are being kept rather than arbitrarily picking one.

### Actual values, not just interesting strings

We also started decoding the model and system information into something readable. The dump contains the Civic Si configuration label, `NR-241UH4CC50-T` model number, stored OS and boot version strings, and filenames for display, camera, hands-free, and voice-related parameter profiles.

There is now a local viewer with 33 decoded fields, nine extracted parameter records, and a byte inspector for everything still unresolved. Some labels are confirmed by the reader code; others remain interpretations. A value looking like a feature flag is not enough to declare that changing it enables something.

The display parameters were a nice example of why tracing matters: the storage capacity is larger than the structure the loader actually reads and checksums. Using the whole allocation as the checksum range would have given us the wrong answer.

### Does this get us closer to native CarPlay?

Yes, although it doesn't settle whether the hardware is compatible.

The CR-V update replaces the SD updater before continuing with the rest of the installation. That explains why the early reboot was such an important boundary in the original failed update. It can also update a separate system controller through a serial download path. That controller payload contains CarPlay-related names, but we haven't established whether the Civic's installed controller already supports the necessary commands.

Importantly, this NOR dump does **not** include that separate controller's firmware. It backs up the targeted main flash, not every chip in the unit.

The next step is to keep tracing what `CarPlay.exe` needs to start, compare those requests with the Civic's existing services, and use small logged tests to identify the actual incompatibilities. We now have a much better baseline for that work, instead of another pile of slightly different boot loops.

The Pi setup remains the working CarPlay option. Native CarPlay is still the experiment. But we can run our own code, inspect the main internal flash without desoldering it, and explain substantially more of the boot process than we could a few days ago. I'll take that.

The raw flash dumps and recovery images are staying private for now. They contain unit-specific data, and I want to keep the working recovery references intact while this continues.
