---
layout: post
title: "Recovering the Honda Civic headunit (for real this time)"
excerpt: "The Civic headunit is working again. Getting there meant fixing a misleading SD capture, figuring out what the CR-V update actually replaced, restoring the original Civic updater without breaking its integrity checks, and using that updater to clear the persistent update state. Here's what worked, what didn't, and what this tells us about the boot process."
date: 2026-10-21
published: false
comments: true
assets: /blog/assets/posts/2026-10-21-recovering-the-honda-civic-headunit-for-real-this-time
categories:
  - reverse-engineering
  - automotive
  - embedded-systems
tags:
  - honda
  - honda-civic
  - windows-ce
  - firmware
  - sd-card
  - reverse-engineering
  - bootloader
  - recovery
---

The Honda radio lives! Again. Hopefully I can keep it that way for a little while this time.

At the end of the last post, I had convinced my 2014 Civic headunit to accept the CR-V CarPlay update. It prompted me to update, I selected Yes, and it actually started copying. Around 4%, it rebooted. Then it kept rebooting. Honda logo, black screen, Honda logo, black screen. Eventually the display would sometimes start doing a very strange color fade, with the outline of the logo still visible, before ending up white or black.

That was where things sat for a while. I could still make it display an error by removing the SD card, so something was alive, but getting back into the radio itself was another story.

We have now recovered it using a modified SD image, without removing the internal flash or using an external programmer. There were quite a few failed images between those two points, though, and some of the things I thought I knew about this system turned out to be wrong.

### First, we needed a clone we could actually trust

The factory SD card would reproduce the boot loop immediately. Some of my freshly flashed spares would instead take several seconds and display `00000006`, which is also the behavior I could get with no card installed.

This made testing incredibly annoying. Did a patch break validation, or was the radio rejecting the spare? At one point even an allegedly stock image produced the error.

I went back to the Teensy 4.1 setup from the earlier SD work and took a fresh capture from the factory card. After fixing a wiring issue, it could initialize and unlock the card again. The actual card reported 7,741,440 sectors, or 3,963,617,280 bytes.

The host script, however, confidently announced that I had an approximately 803 GiB SD card. Nice upgrade, I guess.

There was a serial framing problem. The host had interpreted text as the sector count, and the resulting capture included the real four-byte sector count before the actual card contents. The number `1685217603`, which appeared as the supposed block count, is particularly telling: interpreted as little-endian bytes, it spells `Card`.

The file contained the expected firmware and strings, but every actual SD sector was shifted by four bytes. Flashing software could faithfully write and verify that file, and it would still be wrong.

The correction was to separate the serial protocol header from the SD payload. The original capture was preserved, and a separate aligned image contained exactly the card's reported number of sectors. The first 128 KiB CRC then matched the factory card. More importantly, a spare flashed with that aligned image reproduced the same boot loop as the original.

Finally, a useful baseline.

This also cleared up some confusion about macOS mounting the card. The aligned image does contain a valid MBR and a roughly 272 MiB FAT16 partition labeled `NO NAME`. The interesting Honda data sits outside that partition. The factory card's password protection and whether a reader can expose it are separate issues. A card not appearing on the Mac was not proof that its opening sectors had been deliberately zeroed or that it had no partition table.

### What did the update actually overwrite?

With the capture aligned, I could compare the pre-update backup against the current factory card and the original `SwUpdate.mef` payloads.

This also corrects something from my earlier posts: the SD card contains considerably more of the operating system than I initially realized. There is a large normal-radio OS image, a snapshot image, and a separate, smaller update environment. Finding one `NK.bin` did not mean I had found everything.

Here are the important ranges we compared:

| Component | SD byte offset | Pre-update vs. current |
|---|---|---|
| Normal radio OS / NAVI | `0x12000000` | Unchanged across the compared destination range |
| Snapshot / SSI | `0x1B000000` | Unchanged across the compared destination range |
| Updater / LDA | `0x24000000` | Replaced with the CR-V update's payload |
| DATA resource ranges | `0x25000000`, `0x29C00000` | Unchanged across the compared destination ranges |

The updater was the big finding. The compressed image currently on the card matched the LDA component in `SwUpdate.mef` byte for byte. It was not a partially copied or obviously corrupt gzip file. Its gzip integrity checks and internal `B000FF` record checksums passed.

There were also differences elsewhere in the card's backup/configuration area. I am not calling those update flags just because they changed. The backups were taken at different times, with usage and reboots between them. Those differences do not tell us exactly what happened during the first few percent of the update.

The firmware comparison does show that the main OS and snapshot had not been replaced by the new package in the ranges checked. The unit had successfully replaced its updater, then failed to get through the next stage.

The write boundary was interesting too. The replacement gzip is 2,456,187 bytes, but the changed SD area ends on the corresponding 512-byte sector boundary: 2,456,576 bytes. The old updater's remaining tail beyond that rounded write was still present. That fits a length-limited raw SD write, rather than a filesystem copying a file somewhere.

### That reboot at 4% probably wasn't the crash

I had been treating the reboot at 4% as the point where everything went wrong. However, people doing the intended CR-V update see that reboot too. Their units come back to the update screen and continue.

Looking at the original Civic updater's code made that behavior much easier to understand. Its `NDataUpdateLoadingApp::RunUpdate` path writes the replacement updater, checks the written data, publishes new loading metadata, clears the previous metadata bank, and deliberately restarts the system.

The updater is updating itself before continuing with the rest of the installation.

For the populated fixed components in this package, the recovered order puts LDA before the normal OS and snapshot. Other resource and controller handling exists as well. The observed card contents fit a failure at that updater replacement/restart boundary: the new updater is there, while the main OS is still the old one.

I can't map a particular instruction to the exact frame where the screen said 4%. The percentage is calculated from different kinds of update work; it is not a byte offset into the MEF. We also haven't dumped the internal flash to reconstruct every intermediate write. But the deliberate restart in the Civic code, the CR-V's normal behavior, and the final SD contents all point in the same direction.

I wanted to get the original Civic updater running again, then get it to leave update mode.

### A lot of images that didn't fix anything

The obvious first attempt was to take the old updater from the pre-update backup and put it back at the same SD offset. That produced `00000006`.

We also tried smaller changes rather than replacing the whole environment: transplanting the old `LoadingApp.exe`, inserting holds before update processing, and using artificial progress markers around 37% and 41%. There were tests involving VPU initialization, the added USB module, and changes to the startup timer/watchdog-related constants. A separate hold targeted the normal OS kernel entry.

Most of those produced the same boot loop. One experiment seemed to make the color fade happen immediately, which looked promising, until another run behaved like it always had. So much for that clue.

There were logging attempts too. We could verify the reserved diagnostic sectors through the Teensy, but the radio did not leave a complete checkpoint. That did not prove the executable never ran. A recorder can fail to open its storage device, reject a capacity check, or stop before its first successful write. We had a working way to retrieve a log, but no positive evidence that the in-radio writer had reached that point.

An unchanged result was hard to interpret. A patched DLL inside an image is not evidence that the radio called it. Likewise, a purple screen does not identify a bad display driver. The CR-V updater's exact failure is still unresolved.

### Restoring the old updater without moving it

There was a concrete problem with simply swapping the original gzip back in:

```text
Original Civic updater gzip:  2,517,793 bytes
Installed CR-V updater gzip:  2,456,187 bytes
Difference:                      61,606 bytes
```

The updater replacement code maintains internal loading descriptors containing information such as the image location, length, checksum, and storage device. We could recover that format from the executable, although the installed descriptor values had not been read directly.

If the loader now expected the replacement's shorter length, putting the old, longer gzip at the same address would not be enough. Reading only the expected length would cut off the old stream. Its compressed-data checksum would be different too.

The approach that finally worked was to keep the original Civic updater's decompressed contents intact, while changing its compressed representation to fit the replacement's known constraints.

The original NK was compressed more tightly, leaving room for valid empty DEFLATE blocks. Those blocks contribute bytes to the compressed stream without adding anything to the decompressed output. Their encoding could be adjusted to meet the required size and additive checksum targets.

The finished gzip retained the replacement's 17-byte header, its exact 2,456,187-byte length, a byte sum of `0x133B2B2B`, and a sum over complete little-endian DWORDs of `0x2BCE5AC0`. Its own gzip CRC and size were valid. Standard decompression recovered the exact original Civic NK, with all 66 original record checksums intact.

The image used the full pre-update backup, with the repackaged updater at its original SD address, `0x24000000`.

And it booted differently.

After the Honda logo, I got a Software Update screen saying:

![USB read-error message extracted from the Civic updater]({{ page.assets }}/civic-updater-usb-error.png)

*The message panel extracted from the SD card, rather than a photo of the full screen.*

I power cycled it. Same screen. No boot loop.

I don't think I have ever been that happy to see a USB error.

The full pre-update image also restored old backup/configuration contents. We know this combination worked; we haven't independently proved which individual loader checks made the earlier swaps fail.

### Following the USB error back into the code

That message looked familiar from the bitmap assets found earlier in the project. Sure enough, the exact message is in a compressed SD resource package with a header at `0x24F40000` and gzip data beginning at `0x24F40028`.

The Civic updater reads those resources through `NTelopDraw`. Following the references back led to the display monitor's USB status handling. One confirmed path constructs a filename ending in `Swupdate.mef`, attempts to open it, and selects the USB-error status when opening fails. Other USB errors share the same panel, so the wording alone does not tell us which operation failed.

It did give us a much more useful environment to work in. We had restored the Civic updater far enough to display and handle an error. It still wouldn't return to the radio, even after removing power.

At this point, inserting the update USB was a tempting way to get past the message. I left it disconnected. The recovered updater loop proceeds from finding and validating the package into writing components; I did not find another Yes/No gate in that path. We had already accepted the update back in the normal radio UI.

An accepted package could continue installation, or replace the restored updater again. Which component would be next depended on the saved state. Getting rid of this screen by feeding it the CR-V package would have put us right back into the process we were trying to escape.

### The updater already knows how to clear its state

The better lead was the original updater's successful completion path.

Inside `LoadingApp.exe`, that path calls two routines:

```text
SetLoadingInfo(0, 0, 0, 0)
SetLoadingFlg(0)
```

The first writes an eight-byte status record. The second clears the request DWORD in a larger service-to-boot information table. The recovered driver code places those in logical NOR regions `0x1B0000` and `0x1A0000`, respectively. These are internal-flash driver addresses, not SD offsets.

So restoring the card alone could leave the radio expecting an update. We had given it a working updater, but had not necessarily told it that update mode was finished.

Calling the completion routines immediately at EXE entry would have been too early. They need an opened flash driver, and the request writer uses a cached copy of the handoff table. Calling it without that table could be a very expensive mistake: these routines can erase and rewrite their containing flash sectors.

The hook we used runs after `DiscLoadingReady` succeeds, before the normal package-processing loop starts. It has the original drivers and UI available, but does not enter the component-writing loop.

The new image also bypasses the usual startup status branches and suppresses the known original application calls to these state writers outside the guarded helper. That lets the helper capture the records before those application paths change them.

The recovery logic was roughly:

1. Check that this is the prepared larger spare SD, including its capacity and unique diagnostic controls.
2. Freshly read the actual internal table and status record, and validate their sizes, checksums, and supported status values.
3. Record an attempt marker, then save the original records into reserved sectors on the spare. Read them back and compare before allowing a custom internal write.
4. Clear the status if needed, check the writer's return, and reread the status.
5. Clear the update request if needed, then reread and validate the table. Check that every remaining byte in its declared extent is unchanged.
6. Save and verify the after-clear records, then hold.

Those backups contain the relevant records, not a complete NOR dump. The existing Teensy program can retrieve the diagnostic words and backup-sector CRCs without new firmware; recovering the arbitrary raw backup contents would require a raw capture.

An attempt marker prevents repeated erases if the radio enters the helper again. Already-clear fields are skipped, and the helper does not automatically reboot.

Before testing, the generated SH-4 instructions were exercised against simulated API behavior, including failed reads, failed backups, ignored writes, corrupted readback, and repeat attempts. That doesn't emulate the hardware, but it does catch mistakes in the helper before putting it on the card.

### 0, 65, 66, 77 ... and then the radio came back

For this test, the percent field was repurposed as a diagnostic marker. It was no longer reporting how much firmware had been installed.

I flashed the spare, installed it, and powered on with no update USB connected.

The screen went from 0 to 65, then 66, then 77, and sat there.

65 meant it had entered the status-clear phase. 66 meant it had entered the request-clear phase. 77 meant both records had been read back as clear, the remaining table checked, and the after-clear backups verified.

I waited, turned it off, and turned it back on.

BOOM. The radio was back.

After all of the swapping, recompressing, failed probes, and staring at that stupid Honda logo, it was working again. No chip removal. No external flash programmer. No continuing the CR-V installation.

### Where this leaves the project

We now have positive evidence that modified code in the SD-based Civic updater can execute and use the original internal-flash interfaces. We also know that the existing normal radio software remained usable enough to boot and restore operation after the update state was cleared.

The recovery had two parts: restore a bootable Civic updater environment, then use it to clear persistent update state. This does not tell us which CR-V driver or startup change caused the original loop, or prove that clearing flags alone would have fixed the replacement updater. Both questions still need work.

I have preserved the exact working recovery image, its checksums, the analysis, and a verified compressed backup. The card used in the radio also contains the actual before/after records, which still need to be retrieved. Those are separate from the pristine image saved on the computer.

I am keeping the firmware and recovery SD images private. This worked on my unit in this particular state; I haven't established it as a universal recovery method for every related Honda radio.

Going forward, I want to use the working Civic environment to get reliable checkpoints earlier in startup, inspect the saved update records, and compare the replacement updater's initialization against the original. The old tests often left us asking whether a patch had executed at all. We finally have a way to get a definite answer to that, and a recovery image that has actually brought the radio back.

CarPlay on the original Civic radio is still an open question. For now, I am very happy that the next experiment can start with a working radio.
