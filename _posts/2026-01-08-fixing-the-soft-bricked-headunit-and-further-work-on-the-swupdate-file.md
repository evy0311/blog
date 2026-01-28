---
layout: post
title: "Fixing the soft-bricked headunit, further work on the SwUpdate file, and then bricking it (again)"
excerpt: "After soft bricking a Honda Civic headunit while reverse engineering its update mechanism, I was able to recover it, reverse the firmware gating checks, and ultimately convince the system to accept a modified update package. Along the way, I mapped file validation, checksum enforcement, hardware and version checks, attempted a real update, and pushed the unit into a boot loop state. This post documents the process, the failures, and the insights gained while peeling back how Honda and Mitsubishi designed the SwUpdate pipeline."
date: 2026-01-27 08:27:21 -0500
categories: []
tags: []
assets: /assets/posts/2026-01-08-fixing-the-soft-bricked-headunit-and-further-work-on-the-swupdate-file
published: true
---

> Disclaimer: This post documents reverse engineering work for educational and research purposes. Firmware updates can permanently brick hardware, may be illegal or unsafe in some contexts, and can have safety implications in a vehicle. Do not attempt this unless you fully understand the risks and have a recovery plan.

At the end of the last post, you might recall that I soft bricked the Honda Civic headunit.

That worried me a bit because I was fully prepared to buy a flash programmer and a hot air rework station, remove the NOR flash, dump it, figure out what boot flags were set, reset the state, and re-install the chip.

But before going nuclear, I noticed two important clues:

1. The red flashing light for the anti theft system was still flashing. That suggested the unit was still alive enough to run at least part of its normal logic.
2. When I searched `SwUpdate.mef` for the string `Preparing for program update`, it was present. That string appears to come from the Windows CE side of the system, not a tiny low level boot stub.

### Recovering the soft brick

My theory was simple: I had been booting the unit on my bench with only power. What if the headunit was stuck because it was missing CAN messages that it expects from the car? In particular, I wondered if the anti theft handshake from the original vehicle might be required to exit the state it was in.

So I pulled the Joying radio currently installed in the car, plugged the Honda DA radio back in, and turned the car to ACC.

At first, the headunit behaved the same. I shut the car off, waited about a minute, then went back to ACC and bam: the radio came up immediately and showed the PIN input screen.

That was the whole fix.

If your unit gets stuck in this kind of soft bricked state, a power cycle while connected to the car (so it receives CAN traffic) may be enough to bring it back.

### Back to the SwUpdate.mef work

With the unit recovered, I picked up where we left off: patching the `SwUpdate.mef` update package (originally for a CR V) to be accepted by a Civic headunit.

At a high level, the goal was not just to install a CR V package on a Civic. The bigger win is understanding the update container and the gating checks that decide whether the UI will even show the update prompt. If we can reliably satisfy those checks, then we can run controlled experiments on the update process itself.

A huge advantage here is that Mitsubishi left a factory diagnostic menu on these units.

### Dumping logs from the headunit

The most useful feature for this work is the output log export.

From the radio home screen:

1. Press **Home**, **Menu**, and **Eject** at the same time.
2. Choose **Self diagnosis mode** or **Detail information & Setting** (on my unit it is **Detail information & Setting**).
3. Press and hold **Menu** again for a few seconds.
4. Choose **Output log**.
5. Insert a USB flash drive.
6. Tap **Output Log** and wait a few seconds.

Pull the USB drive, plug it into a computer, and locate `SER_MSG.LOG`.

That file is gold. It tells you exactly which check failed and which detect result code the loader returned.

### The gating checks

From repeated tests and log captures, there are a few distinct gates that happen before you ever see a real update screen.

The names below come directly from the log output.

#### Gate 1: File discovery and naming

The loader looks specifically for a file named `SwUpdate.mef` on the USB.

If the file is missing or renamed (for example `SwUpdate2.mef`), the logs show an open failure and then a detect result that corresponds to no update file found.

Example log pattern:

```text
... UILoading_NaviAutoLoadingDetectorHONDA::DetectAutoLoading()
... NFile::OpenFile(\UH\SwUpdate.mef) ...
... UILoading_NaviAutoLoadingDetectorHONDA::Update() # => DetectResult [2]
```

So the very first gate is not magic header parsing. It is a boring, strict file name check.

#### Gate 2: Basic header sanity checks

Next, the headunit reads a tiny chunk (often `Sz=4`) and then reads the first 512 bytes.

This is where basic structure checks happen, including a critical one: the 32 bit little endian file size stored in the header must exactly match the actual file size.

From our offset flipping tests, bytes `0x14` through `0x17` are the stored size. If that size is wrong, the unit bails early with a distinct detect result.

This is a classic gate because it is fast, cheap, and immediately rules out corrupted or partially copied files.

#### Gate 3: Ability file checksum

Once the file passes naming and basic header sanity, the next hard stop is:

```text
UILoading_UpdateDataFile::Check(): AbilityFile CheckSum Error.
... => DetectResult [4]
```

This was the main wall for a long time.

The logs are a little misleading if you read them as "bytes from the beginning of the file". `Sz=886` means it read 886 bytes from some region, not necessarily the first 886 bytes.

The key realization was that the loader appears to be validating a specific configuration blob, and that blob is what the logs call the ability file. In the CR V package we were using, the region we labeled `MODULE_0` fits inside that 886 byte read window.

In other words, `MODULE_0` is not just some random record. It is very likely the ability config that is hashed or checksummed.

Once we understood that, the path forward was to:

- Identify the exact byte coverage of the checksum (for example, whether it stops at the `END` marker or covers padded bytes past it)
- Recompute the checksum correctly after any edits
- Patch the file so the stored value and the computed value match

After iterating through this and validating the algorithm, we were able to produce a `SwUpdate.mef` that passed the ability file checksum gate.

#### Gate 4: Hardware name match

The next gate, once the ability file is accepted, is hardware identity.

The log call looks like this:

```text
UILoading_AbstractAbilityConfigFile::CheckHardwareName => HardWareName Error
... => DetectResult [10]
```

This is a compatibility check that prevents applying a package intended for different hardware.

In practice, this gate is string based. The update package contains one or more hardware name identifiers (the ones we kept seeing were around the same string region as `H15M` and `Sasanqua`).

In the CR V update we started from, the manifest region referenced `Sasanqua_iUS`, while our Civic unit identifies as `Sasanqua_US`. Even stranger, many places inside the package still reference `Sasanqua_US` without the extra `i`.

Regardless of why Honda shipped that inconsistency, the loader only cares that the expected hardware name matches what the headunit reports.

After patching the hardware name fields so they match the Civic unit, the loader finally stopped throwing DetectResult [10].

#### Gate 5: Application version check

After the hardware name gate, the loader performs an application version comparison.

The log output looks like this:

```text
CheckAppVersion => AppVersion OldError
```

This appears to be a monotonic version check that prevents downgrades or installing an update that advertises an application version lower than what is currently installed.

In practice, this means the version fields embedded in the update package must be greater than or equal to the versions reported by the running system. If they are not, the loader exits early and the UI never presents the update option.

This gate is separate from the hardware check and confirms that version compatibility is enforced independently.

#### Gate 6: Build version validation

Closely related to the application version gate is a build version check.

When this fails, the logs indicate a build version error rather than an ability or hardware failure. This suggests the loader validates both a human visible application version and a lower level build identifier.

Both values must be acceptable for the update to proceed.

At this point in the process, the file has already passed naming, header validation, ability checksum, hardware name matching, and now version validation. Only after all of these gates succeed does the headunit treat the file as a legitimate update.

### The payoff: the real update screen

After we fixed the size header, the ability file checksum, and the hardware name mismatch, the headunit finally presented the proper update UI.

That was the milestone we needed.

It proves that the gating checks are not a cryptographic signature over the whole update package (at least not at this stage of the process). Instead, the loader appears to use a sequence of targeted sanity checks and compatibility checks before it ever shows the menu.

![Real update screen](https://github.com/evy0311/blog/raw/gh-pages/assets/posts/2026-01-08-fixing-the-soft-bricked-headunit-and-further-work-on-the-swupdate-file/civic_real_update_screen.jpeg)

I am willing to bet I may be the only person to have ever seen this screen appear on this model of Honda DA radio, shared with the 2014-2015 Civic, and similar year Fit, HRV, etc. Honda never released any official updates for these radios, other than maps updates. So this may be the first time one in the wild has seen this screen. Awesome!

### Attempting the update

Once all known gates were satisfied, the headunit finally recognized the USB as containing a valid update and prompted me to proceed.

At this point, I knew there was a high likelihood the update would fail. The CR V radio and Civic radio share a very similar architecture and processor family, but there are real hardware differences at the board and peripheral level. Display panels, timing controllers, and related drivers are common failure points in cross device firmware experiments like this.

Still, the entire exercise was about learning and exploration, so I went ahead and attempted the update.

The unit rebooted as expected and began the update process. Shortly after, it rebooted again and entered a persistent boot loop.

My current working theory is that this is caused by a display driver or timing mismatch. The core processor and OS are likely intact, but the system may be failing during early UI or display initialization.

Below is a video showing the update attempt and the resulting boot loop:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe
    src="https://www.youtube.com/embed/htbw9guPRdk"
    title="2014 Honda Civic Headunit - Attempting Carplay software update"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
    allowfullscreen
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;">
  </iframe>
</div>

### What this means for future work

Now that we can reliably reach the update prompt, we can do controlled experiments:

- Change one field at a time and confirm exactly which gate it trips
- Map each detect result code to a specific failure path
- Correlate file reads to specific regions in the container
- Explore what happens after the UI accepts the package (the part that is more likely to involve deeper integrity checks)

In the next post, I want to dig into the second `NK.bin` we found inside the package, since it contains a lot of the files that appear to drive the visible UI and logic of the headunit.

For now, the key takeaway is that logs plus disciplined byte level experiments are enough to break the process into understandable gates.

This outcome was not entirely unexpected. From the start, this was never about successfully converting a CR V radio into a Civic radio. It was about understanding the update pipeline, the checks involved, and the boundaries enforced by the system.

Reaching the point where the headunit fully accepts an update package is a major milestone on its own. Everything after that point is deeper system territory, where hardware specific drivers and early boot configuration matter far more than container format correctness.

### A note on tooling and responsibility

At this stage, I do have a working script that can patch the update package so it passes all known gating checks and is accepted by the headunit.

I am intentionally not sharing that script yet. The risk of someone attempting this without fully understanding the implications and permanently bricking their own hardware is very real. Until I better understand recovery paths and the exact failure mode here, keeping that tooling private is the responsible choice.

### A surprising SD card discovery

While examining the SD card contents used by the headunit, I discovered another hidden ROM image containing a much larger portion of the actual operating system.

This ROM includes many of the same binaries we previously extracted from `SwUpdate.mef`, including `Diag.exe`, which contains much of the logic responsible for update detection and validation.

Finding these files at rest on the SD card confirms that our reverse engineering work on the update package was targeting real, live system components, not just installer stubs.

### Where things stand now

The headunit is currently in a boot loop state, but the system is not beyond recovery. The earlier soft brick demonstrated that the unit is resilient and that recovery paths exist.

The next phase of this project is focused on understanding the boot sequence in more detail, identifying where the failure occurs, and determining whether the system can be recovered through CAN interaction, SD card manipulation, or external flashing.

Even with the current boot loop, this represents significant progress toward understanding how these systems work. This project remains what it has always been: a learning exercise, a technical challenge, and frankly, a bit of fun.

### Appendix: A tiny helper for controlled bit flips

For completeness, here is the little helper script I used to flip a single bit at a chosen offset, so we can provoke different failure codes without making huge edits.

```python
#!/usr/bin/env python3
import sys

def main():
    if len(sys.argv) != 4:
        print("usage: flip1.py in_file out_file hex_offset")
        print("example: flip1.py SwUpdate.mef SwUpdate.patched.mef 0x136")
        return 2

    in_path, out_path, off_s = sys.argv[1], sys.argv[2], sys.argv[3]
    off = int(off_s, 16) if off_s.lower().startswith("0x") else int(off_s)

    b = bytearray(open(in_path, "rb").read())
    if off < 0 or off >= len(b):
        raise SystemExit(f"offset out of range: 0x{off:x} (file size 0x{len(b):x})")

    old = b[off]
    b[off] = old ^ 0x01  # flip lowest bit

    with open(out_path, "wb") as f:
        f.write(b)

    print(f"wrote {out_path}")
    print(f"offset 0x{off:x}: {old:02x} -> {b[off]:02x}")

if __name__ == "__main__":
    raise SystemExit(main())
```