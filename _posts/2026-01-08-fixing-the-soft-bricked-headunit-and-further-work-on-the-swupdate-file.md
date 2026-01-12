---
layout: post
title: "Fixing the soft-bricked headunit and further work on the SwUpdate file"
excerpt: ""
date: 2026-01-08 08:27:21 -0500
categories: []
tags: []
assets: /assets/posts/2026-01-08-fixing-the-soft-bricked-headunit-and-further-work-on-the-swupdate-file
published: false
---

TODO: Add disclaimer about actions taken

At the end of the last post, you might recall that I soft bricked the Honda Civic headunit. This worried me slightly because I was fully prepared to buy a T48 and hot air rework station to pull off the NOR flash chip, dump it, reset the boot
flag state it was in, and then re-install. But then I had a theory. In the soft bricked state, I noticed two things:
1. The red flashing light for the anti theft system was still flashing. This made me think that maybe the radio was still "alive" and in a state where it would be processing these things.
2. When I searched SwUpdate.mef file for "Preparing for program update" it was there, which means this was coming from the actual Windows code, not some low level bootloader setup. 

My wild theory was this: I had been botting it on my bench with just power, no other connections. What if I reconnected it to the car, so that it could start sending and recieving CAN data from the car, could this bring it back to life?
My thought was that possibly, the anti-theft message sent from the original car its mated to, might be enough to snap it out of the state it was in. So, I removed the Joying radio I currently have installed in the car,
plugged in the Honda DA radio, and turned it to ACC mode. THe radio behaved the same. So I turned off the car, let it sit for a minute, then went back to ACC mode and bam! The radio instantly came up and showed the PIN input screen, and all is back to normal.

This is good to know: if the radio gets stuck in this state, a power cycle in a setting that sends it CAN messages is likely enough to fix.


Now that we are back in business, its time to continue where we left off. The last time I piosted, I was attempting to modify the SwUpdate.mef file that Honda released for the CR-V radio to work in the Civic radio. As far as I understand
these radios are nearly identical. If I brick the unit trying, atleast I tried. So on we go! 

My next step was to try and patch the SwUpdate.mef file to see if I could get my Civic radio to recognize it as an update. Even if it wouldnt be compatible, understanding the mef file structure and overall logic as it pertains to the radio update process
would allow us to potentially make other changes to the radio, like allowing video in motion, disabling the initial "The driver is responsible" warning, and other things. While these are all to be done at ones own risk, it is important to remember that the purpose
of this project overall is just for research and educational purposes.

One great thing that Mitsubishi left us in the radio was a hidden admin/debug menu that allows for factory diagnostic functions. The most important to us currently is the "Output log" feature. To access, from the radio home screen,
press the "Home", "Menu" and "Eject" buttons at the same time. After a few seconds, a screen will come up with two options: Self-diagnosis mode, and detail information & Setting. Pick detail information & setting. From here, press and hold 
the "Menu" button again for a few seconds. That brings up a new menu. The 5th option down on my radio is "Output log". Press that. Then a new screen will come up with a single button, reading "Output Log". Ensure you have a USB flash drive plugged 
in to either of the two ports. When the button is enabled, click it. It will take a few seconds. Then, remove the drive and connect to a computer. TODO: Finish





TODO: Talk about the NEW NK.bin file i found and extracted! 
I decided to do some further digging of the SwUpdate file at this point because we were still not seeing the DetectResult[5] origin in our original NK.bin. After much digging, I found that there was ANOTHER NK.bin file that exists within our SwUpdate.mef file,
and it was completely missed by binwalk. I was able to extract this NK.bin file from our previous nk_inf.bin file (which was extracted from the overall SwUpdate.mef file based on the beginning start and end addresses in the manifest region).
After I extracted this, it ran cleanly through dumprom.exe and gave us a TON of new files. The most exciting part of this, is that all of the files here appear to be the same exact things that run on the radio that the user sees. All of the actual UI
and functionality is here, and the files are VERY telling. Lets dig in..

Thankfully for us, there is a specific log file that seems to be a general running log of the radio. Named "SER_MSG.LOG", this log file contains a lot of important information as it pertains to our update file. This is under the folder named with a bunch of numbers, likely a date code.
If we follow the process of patching our SwUpdate.mef file, plugging it into the radio, and getting the error, we can then go and dump the logs to see why it failed. If we utilize the file as it is from the CR-V, we get some error logs like this:

```text
PID:049F0026 TID:05310192 0000139415 @@@ VersionInfo Name:Sasanqua_US Version:1.D200.30
PID:049F0026 TID:097D000A 0000139417 NPerformMeasur:Fin [ResLate]Th=0x097D000A F="\UH\SwUpdate.mef" Sz=512 Tick[S=139382 E=139417] ThT[S=6 E=7]
PID:049F0026 TID:097D000A 0000139442 0000139442:# UILoading_UpdateDataFile::Check(): AbilityFile CheckSum Error.
PID:049F0026 TID:097D000A 0000139443 0000139443:# UILoading_NaviAutoLoadingDetectorHONDA::Update() # => DetectResult [4] 
PID:049F0026 TID:04BA0026 0000139460 0000139460:# UILoading_SystemStatusManager::ResetLoadingSourceStatus() #
```

and then
```text
PID:049F0026 TID:097D000A 0000716044 NPerformMeasur:Fin [ResLate]Th=0x097D000A F="\UH\SwUpdate.mef" Sz=4 Tick[S=716021 E=716044] ThT[S=20 E=22]
PID:049F0026 TID:096C0476 0000716073 @@@ VersionInfo Name:Sasanqua_US Version:1.D200.30
PID:049F0026 TID:097D000A 0000716097 >>>>>>>>>>>>>> UILoading_AbstractAbilityConfigFile::CheckHardwareName => HardWareName Error
PID:049F0026 TID:097D000A 0000716098 0000716098:# UILoading_NaviAutoLoadingDetectorHONDA::Update() # => DetectResult [10] 
PID:049F0026 TID:04BA0026 0000716112 0000716112:# UILoading_SystemStatusManager::ResetLoadingSourceStatus() #
```

Of course that is just a snippet of the entire log file (with some other info around it about mounting the USB, etc). But this gives us a good idea on whats happening. This is just theoretical, but we can see it read 512 bytes (Sz=512),
threw an AbilityFile checksum error (Detect Result = 4), and then read 5 bytes (Sz=4) and threw a HardWareName Error (Detect Result = 10). 

The odd thing is I have seen that behavior, but other times, with the same file, I have seen it log

```text
PID:049F0026 TID:04BA0026 0000100853 0000100853:# UILoading_NaviAutoLoadingDetectorHONDA::DetectAutoLoading() 
PID:049F0026 TID:084F000E 0000100893 NPerformMeasur:Fin [ResLate]Th=0x084F000E F="\UH\SwUpdate.mef" Sz=512 Tick[S=100879 E=100893] ThT[S=5 E=6]
PID:049F0026 TID:09EB00F6 0000100908 >>>>>GetLogThread:		 PROC:049F0026 ThreadID:09eb00f6
PID:049F0026 TID:09EB00F6 0000100912 @@@ VersionInfo Name:Sasanqua_US Version:1.D200.30
PID:049F0026 TID:084F000E 0000100913 NPerformMeasur:Fin [ResLate]Th=0x084F000E F="\UH\SwUpdate.mef" Sz=928 Tick[S=100895 E=100913] ThT[S=8 E=10]
PID:049F0026 TID:084F000E 0000100929 >>>>>>>>>>>>>> UILoading_AbstractAbilityConfigFile::CheckHardwareName => HardWareName Error
PID:049F0026 TID:084F000E 0000100929 0000100929:# UILoading_NaviAutoLoadingDetectorHONDA::Update() # => DetectResult [10] 
```

Indicating that it read 512 bytes, then 928 bytes, and then threw the HardWareName error. It is possible that there is some sort of caching going on, I suppose. We can see there are two different checks/processes at play here, though. The first
is `UILoading_UpdateDataFile::Check()` which is the one throwing `AbilityFile CheckSum Error (DetectResult [4])`. The second is `UILoading_AbstractAbilityConfigFile::CheckHardwareName`, which is the one throwing `HardWareName Error (DetectResult [10])`.

The hardware name error should be easy enough to get past, as that is a patch on the `HW` strings in the manifest. The interesting thing is, they are set to `Sasanqua_iUS` in the file for the CR-V. But, that platform and the Civic radio both are actually `Sasanqua_US`. Even in the depths of this
specific SwUpdate.mef file from Honda, everything is referred to as `Sasanqua_US`, with no i in the string at all. I'm not sure why that is, but maybe we will run into it later. It is worth noting. 

TODO: Talk about wifi and oither theoretical functions
TODO: Talk about exe files, font, registry etc