---
layout: post
title: "Attempting to fix the boot loop state after update attempt"
excerpt: ""
date: 2026-01-28 09:20:09 -0500
categories: []
tags: []
assets: /assets/posts/2026-01-28-attempting-to-fix-the-boot-loop-state-after-update-attempt
---

Post content goes here, dude!

TODO: These are just notes below  

So after examining the pre-update OG SD card image, and the SD card image POST update (failure), there is only 1 single difference between the 2 images: The NK.bin at offset 0x24000000. This is presumably the "boot" NK.bin
because inside, we know it was files like LoadingApp.exe and a few other things. This is PROBABLY what is crashing in the new one. 

Pre update:
603979776                          0x24000000                         gzip compressed data, original file name: "NK.bin", operating system: NTFS filesystem (NT), timestamp: 2013-10-17 08:55:34, total size: 2517793 bytes

Post update:
603979776                          0x24000000                         gzip compressed data, original file name: "NK.bin", operating system: FAT filesystem (MS-DOS, OS/2, NT/Win32, timestamp: 2014-11-25 02:05:34, total size: 2456187 bytes

THey are the same offset, but their size and timestampe is slightly different. This makes sense, as the post update one has a newwer timestamp. Unsure, but it may have to do with the FAT vs NTFS filesystem? 
It is porbably not the issue, NTFS just means it was built with windows NT tooliing, and FAT means it was built with older or cross platform tooling  So it likely is not the issue. 

Next step is to compare: what are the exact differences between the pre update SD card image, and the post update sd card image. its possible there are some hidden changes I am not seeing  

evanhorsley@Evans-MacBook-Pro-2 sd card sniffing % cmp -l civic_dump/civic_dump_raw.img civic_updated_sd_dump/update_civic_dump.img | head

python3 - <<'PY'
import sys
a=open("civic_dump/civic_dump_raw.img","rb").read()
b=open("civic_updated_sd_dump/update_civic_dump.img","rb").read()
n=min(len(a),len(b))
diff=[]
i=0
while i<n:
if a[i]!=b[i]:
j=i
while j<n and a[j]!=b[j]:
j+=1
diff.append((i,j))
i=j
else:
i+=1
print("diff ranges:", len(diff))
for s,e in diff[:50]:
print(hex(s), hex(e), "len", e-s)
PY
18437   0 377
18438   0 377
18439   0 377
18440   0 377
18441   0 377
18442   0 377
18443   0 377
18444   0 377
35845   0 377
35846   0 377
diff ranges: 326008
0x4804 0x480c len 8
0x8c04 0x8c0c len 8
0xd000 0xd002 len 2
0xd003 0xd004 len 1
0xd005 0xd006 len 1
0xd007 0xd008 len 1
0xd009 0xd00a len 1
0xd00b 0xd00c len 1
0xd00d 0xd00f len 2
0xd010 0xd011 len 1
0xd012 0xd013 len 1
0xd014 0xd015 len 1
0xd016 0xd017 len 1
0xd01c 0xd02c len 16
0xd02d 0xd034 len 7
0xd036 0xd03b len 5
0x11000 0x1100c len 12
0x1100d 0x11014 len 7
0x11016 0x1101b len 5
0x11020 0x1102c len 12
0x11030 0x11031 len 1
0x11032 0x11033 len 1
0x11036 0x1103a len 4
0x11040 0x11042 len 2
0x11045 0x1104c len 7
0x1104d 0x1105a len 13
0x1105c 0x11062 len 6
0x11063 0x11064 len 1
0x11065 0x11066 len 1
0x11067 0x11068 len 1
0x11069 0x1106a len 1
0x1106b 0x1106c len 1
0x1106d 0x1106f len 2
0x11070 0x11071 len 1
0x11072 0x11073 len 1
0x11074 0x11075 len 1
0x11076 0x11077 len 1
0x11078 0x11079 len 1
0x1107c 0x1107d len 1
0x1107e 0x1107f len 1
0x11080 0x1108c len 12
0x1108d 0x11094 len 7
0x11096 0x1109b len 5
0x1109c 0x1109d len 1
0x110a0 0x110a2 len 2
0x110a3 0x110a4 len 1
0x110a5 0x110a6 len 1
0x110a9 0x110ac len 3
0x110ad 0x110ba len 13
0x110bc 0x110c2 len 6

Then we run a heatmap on it
Total differing bytes: 3728671

Top 30 sectors by changed bytes (sector_idx, changed_bytes, byte_offset_hex):
1179653     512  off=0x24000a00
1179654     512  off=0x24000c00
1179659     512  off=0x24001600
1179660     512  off=0x24001800
1179663     512  off=0x24001e00
1179666     512  off=0x24002400
1179668     512  off=0x24002800
1179685     512  off=0x24004a00
1179694     512  off=0x24005c00
1179699     512  off=0x24006600
1179704     512  off=0x24007000
1179707     512  off=0x24007600
1179716     512  off=0x24008800
1179728     512  off=0x2400a000
1179732     512  off=0x2400a800
1179738     512  off=0x2400b400
1179743     512  off=0x2400be00
1179744     512  off=0x2400c000
1179750     512  off=0x2400cc00
1179751     512  off=0x2400ce00
1179764     512  off=0x2400e800
1179765     512  off=0x2400ea00
1179766     512  off=0x2400ec00
1179767     512  off=0x2400ee00
1179769     512  off=0x2400f200
1179792     512  off=0x24012000
1179798     512  off=0x24012c00
1179827     512  off=0x24016600
1179834     512  off=0x24017400
1179846     512  off=0x24018c00

Top 30 MiB blocks by changed bytes (block_idx, changed_bytes, byte_offset_hex):
577   1044452  off=0x24100000
576   1044359  off=0x24000000
578    357986  off=0x24200000
1720     19156  off=0x6b800000
1588     19154  off=0x63400000
1592     19149  off=0x63800000
1596     19147  off=0x63c00000
1756     19145  off=0x6dc00000
1696     19142  off=0x6a000000
1712     19142  off=0x6b000000
1724     19142  off=0x6bc00000
1600     19141  off=0x64000000
1716     19141  off=0x6b400000
1604     19140  off=0x64400000
1700     19140  off=0x6a400000
1708     19138  off=0x6ac00000
1608     19137  off=0x64800000
1760     19135  off=0x6e000000
1612     19133  off=0x64c00000
1584     19131  off=0x63000000
1704     19131  off=0x6a800000
1616     19130  off=0x65000000
1764     19130  off=0x6e400000
1624     19129  off=0x65800000
1732     19129  off=0x6c400000
1620     19127  off=0x65400000
1728     19127  off=0x6c000000
1768     19127  off=0x6e800000
1628     19121  off=0x65c00000
1504     19120  off=0x5e000000


Then we run diffs
python3 - <<'PY'
import os, binascii, hashlib, math

OG = "civic_dump/civic_dump_raw.img"
POST = "civic_updated_sd_dump/update_civic_dump.img"

OFFSETS = [
0x6b800000, 0x63400000, 0x63800000, 0x63c00000, 0x6dc00000,
0x6a000000, 0x6b000000, 0x6bc00000, 0x64000000, 0x6b400000,
0x64400000, 0x6a400000, 0x6ac00000, 0x64800000, 0x6e000000,
0x64c00000, 0x63000000, 0x6a800000, 0x65000000, 0x6e400000,
0x65800000, 0x6c400000, 0x65400000, 0x6c000000, 0x6e800000,
0x65c00000, 0x5e000000,
]

BLOCK = 1024 * 1024  # 1 MiB

def read_block(path, off, n):
with open(path, "rb") as f:
f.seek(off)
return f.read(n)

def shannon_entropy(data: bytes) -> float:
if not data:
return 0.0
counts = [0]*256
for b in data:
counts[b] += 1
ent = 0.0
ln = len(data)
for c in counts:
if c:
p = c/ln
ent -= p * math.log2(p)
return ent

def first_ascii_strings(data: bytes, minlen=6, limit=12):
out=[]
cur=[]
for b in data:
if 32 <= b <= 126:
cur.append(b)
else:
if len(cur) >= minlen:
out.append(bytes(cur).decode("ascii", "ignore"))
if len(out) >= limit:
break
cur=[]
if len(out) < limit and len(cur) >= minlen:
out.append(bytes(cur).decode("ascii", "ignore"))
return out

def hex_preview(data: bytes, n=32):
return binascii.hexlify(data[:n]).decode()

def pct(byteval, data: bytes) -> float:
return 100.0 * data.count(byteval) / len(data) if data else 0.0

print(f"OG   : {OG}")
print(f"POST : {POST}")
print(f"OG size  : {os.path.getsize(OG):,} bytes")
print(f"POST size: {os.path.getsize(POST):,} bytes")
print()

for off in OFFSETS:
og = read_block(OG, off, BLOCK)
po = read_block(POST, off, BLOCK)

# changed bytes count
changed = sum(1 for a,b in zip(og,po) if a!=b)

# fingerprints
og_md5 = hashlib.md5(og).hexdigest()
po_md5 = hashlib.md5(po).hexdigest()

# entropy + fill ratios
og_ent = shannon_entropy(og)
po_ent = shannon_entropy(po)
og_ff = pct(0xFF, og); og_00 = pct(0x00, og)
po_ff = pct(0xFF, po); po_00 = pct(0x00, po)

# quick strings sniff
strs = first_ascii_strings(po)

print(f"off=0x{off:08x} changed={changed:6d}  og_md5={og_md5[:8]} post_md5={po_md5[:8]}  ent(og)={og_ent:.2f} ent(post)={po_ent:.2f}  %00 og={og_00:5.1f} post={po_00:5.1f}  %ff og={og_ff:5.1f} post={po_ff:5.1f}")
print(f"  og head: {hex_preview(og)}")
print(f"  po head: {hex_preview(po)}")
if strs:
print("  post strings:", "; ".join(strs[:8]))
print()
PY

Result:
OG   : civic_dump/civic_dump_raw.img
POST : civic_updated_sd_dump/update_civic_dump.img
OG size  : 3,963,617,280 bytes
POST size: 3,963,617,280 bytes

off=0x6b800000 changed= 19156  og_md5=ab064b99 post_md5=b0c88b44  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x63400000 changed= 19154  og_md5=dd7fcddb post_md5=f001a425  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x63800000 changed= 19149  og_md5=b847642b post_md5=9e4dea89  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x63c00000 changed= 19147  og_md5=100f3c13 post_md5=79a64b17  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6dc00000 changed= 19145  og_md5=1333bef1 post_md5=6983ed2a  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6a000000 changed= 19142  og_md5=be064991 post_md5=3c6a5a4a  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6b000000 changed= 19142  og_md5=66d9a9ca post_md5=b0aa6301  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6bc00000 changed= 19142  og_md5=deccced8 post_md5=a4feb3fb  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x64000000 changed= 19141  og_md5=93b00b93 post_md5=828bf1b1  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6b400000 changed= 19141  og_md5=3d68a39e post_md5=e399554d  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x64400000 changed= 19140  og_md5=6dc8e166 post_md5=508b5eaa  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6a400000 changed= 19140  og_md5=57e1a0ab post_md5=3c22c824  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6ac00000 changed= 19138  og_md5=8eb39a30 post_md5=80abc1d8  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x64800000 changed= 19137  og_md5=2fdc80f2 post_md5=1eb223e9  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6e000000 changed= 19135  og_md5=8a41c44f post_md5=ac35b141  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x64c00000 changed= 19133  og_md5=5827c125 post_md5=66159d5f  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x63000000 changed= 19131  og_md5=5dbfb549 post_md5=a683b1e7  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6a800000 changed= 19131  og_md5=0d08de36 post_md5=0b394d80  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x65000000 changed= 19130  og_md5=f091dd3e post_md5=cd0ddf3e  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6e400000 changed= 19130  og_md5=b511eb9c post_md5=780da8ee  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x65800000 changed= 19129  og_md5=5f99776b post_md5=0e63f690  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6c400000 changed= 19129  og_md5=9b324144 post_md5=b84dcf4b  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x65400000 changed= 19127  og_md5=d7fb3394 post_md5=9ba28ce2  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6c000000 changed= 19127  og_md5=2872152e post_md5=772dff08  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x6e800000 changed= 19127  og_md5=321e7dc5 post_md5=bf2fc03f  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x65c00000 changed= 19121  og_md5=7dd8fe9d post_md5=68c7b642  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

off=0x5e000000 changed= 19120  og_md5=06f05a7b post_md5=bb397f96  ent(og)=1.42 ent(post)=1.44  %00 og= 45.1 post= 45.7  %ff og= 51.4 post= 50.7
og head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
po head: ffffffffffffffffffffffffffffffff008000008000000001000000ffffffff
post strings: H14M_DA_KA_BKUPTBL; ClockWallpaper_H; ITUNE_TAGGING_INFO; UI_PLAYER_SECURE4; UI_TUNER_SECURE; UI_DIAG_SECURE3; HMI_VR; SDDM_DATA

We know now that not just NK.bin changed. diff output is showing image level changes in multiple 4 MB blocks at a bunch of different absolute offsets, so it’s not only NK.bin that changed.

What changed is probably
•	NK.bin (the payload you care about), plus
•	one or more system metadata blobs that track:
•	update history / timestamp
•	per-partition or per-slot checksums/hashes
•	“backup table” state (your string H14M_DA_KA_BKUPTBL is a big hint)
•	possibly journal or “secure UI” containers (those UI_*_SECURE string hits are consistent with packed resources or a catalog file)


The frankenstein NK bin from the og card onto tthe img from the post update card did not work, resulted in the same 0000006 code behavior 


Next best candidates, one change per attempt:
1.	Candidate 2: same NK swap, but zero pad the remainder of the window
Sometimes those trailing bytes are expected to be empty after gzip end, and the POST image has nonzero data that now conflicts.
2.	Candidate 3: swap the entire fixed window 0x24000000 to 0x24F40028 from OG into POST
This is the most important next attempt. It preserves whatever per-bundle metadata lives after the gzip stream and keeps it consistent with OG NK.
3.	Candidate 4: swap NK window plus the next gzip blob at 0x24F40028
That second gzip at 0x24F40028 (32,875 bytes) looks like a companion chunk. If the bootloader checks a “set” of blobs, you need them to match.


We then used a make franken candidates script to create those

[+] NK offset: 0x24000000
[+] Window end: 0x24F40028 (size 15990824 bytes)
[+] OG NK gzip len: 2517793
[+] POST NK gzip len: 2456187
[+] OG blob2 gzip len at 0x24F40028: 32875
[+] Wrote: civic_updated_sd_dump/update_civic_dump_C2_NKswap_zeropad.img
[+] Wrote: civic_updated_sd_dump/update_civic_dump_C3_fullwindow_OG.img
[+] Wrote: civic_updated_sd_dump/update_civic_dump_C4_window_plus_blob2_OG.img
[+] Done. No input images were modified.

update_civic_dump_C2_NKswap_zeropad.img had same behavior as candidate 1 before ,jus NK swap


Candidate 5: This reads OG and POST, and for each changed block except 144, it replaces every differing byte with OG’s byte. Output is a new image.
python3 - <<'PY'
from pathlib import Path

OG   = Path("civic_dump/civic_dump_raw.img")
POST = Path("civic_updated_sd_dump/update_civic_dump.img")
OUT  = Path("civic_updated_sd_dump/update_civic_dump_C5_postNK_ogSmallMeta.img")

BLOCK_SIZE = 0x400000

# paste your changed block indices here (including 144, we will skip it below)
changed_blocks = [
0, 144,
376,377,378,379,380,381,382,383,384,385,386,387,388,389,390,391,392,393,394,395,
396,397,398,399,400,401,402,403,404,405,406,407,408,409,410,411,412,413,414,415,
416,417,418,419,420,421,422,423,424,425,426,427,428,429,430,431,432,433,434,435,
436,437,438,439,440,441,442
]

print("[+] Copying POST to OUT...")
OUT.write_bytes(POST.read_bytes())

og_f   = OG.open("rb")
post_f = POST.open("rb")
out_f  = OUT.open("r+b")

def patch_block(bi: int):
base = bi * BLOCK_SIZE
og_f.seek(base);   og = og_f.read(BLOCK_SIZE)
post_f.seek(base); po = post_f.read(BLOCK_SIZE)

    if og == po:
        return 0

    # patch only bytes that differ
    diff_count = 0
    ba = bytearray(po)
    mv_og = memoryview(og)
    mv_po = memoryview(po)
    for i in range(BLOCK_SIZE):
        if mv_po[i] != mv_og[i]:
            ba[i] = mv_og[i]
            diff_count += 1

    out_f.seek(base)
    out_f.write(ba)
    return diff_count

total_patched = 0
for bi in changed_blocks:
if bi == 144:
print("[=] Skipping block 144 (NK block) to keep POST NK")
continue
n = patch_block(bi)
total_patched += n
print(f"[+] Patched block {bi}: {n} bytes")

og_f.close()
post_f.close()
out_f.close()

print(f"[+] DONE. Total patched bytes: {total_patched}")
print(f"[+] Wrote: {OUT}")
PY


What that fade sequence actually means (important)

That sequence is almost certainly:
1.	Bootloader → loads NK
2.	WinCE kernel starts
3.	Display driver initializes
4.	Graphics subsystem partially comes up
5.	Something critical fails after GPU/display init but before UI task loop
6.	System halts or spins with backlight on

This is not an SD read failure.
This is not a checksum failure of NK.
This is runtime failure inside the OS or a driver init path.

If it were a checksum or header mismatch, you’d get:
•	immediate reboot
•	immediate error code
•	or no display init at all

Instead, you get:
•	splash
•	color ramp
•	frame buffer artifacts
•	backlight stuck on

That’s classic “OS started, died mid-init”.


I zeroed out everything from the beginning after the DOS MBR up until the gzip NK.bin, and got the 00000006 error. So it must be reading that or something during boot... could be valuable to us
ACTUALLY, that test may not have been vlaid  I was using that 32gb samsung card and i think that was causing issues. Even booting the known good dump image that causes the boot loop produced
the 00000006 error, so i think its checking card size too maybe? 

INTERESTING... the behavior with everything zerod out between MBR and the gzip NK is the same as with no changes... so those files are not read during the current boot state. This is good (and bad)

Next test was to change only one bit inside gzip header MTIME. No recompression, no size change, no NK payload change. This resulted in the same boot loop behavior, meaning the check is not on the header.

Next we flip 1 byte inside the gap between the gzip member and the next thing binwalk found. This sresulted in the same boot loop behavior. That means the check on "do i accept or boot this card" is likely the gzip member itself,
or the decompressed NK.bin. 

Next we flip 1 bit in the gzip trailer (CRC32 or ISIZE). That resulted in the 00000006 error.

Next take the known-good bootlooping image, decompress NK, patch one string, then recompress BUT ensure we write a standards-compliant gzip member with correct CRC/ISIZE. This resulted in 00000006 error.

What this all means?
Given your experiments:
•	Header mtime flip: still boot-loops (so header fields are not what’s being trusted)
•	Gap flip: still boot-loops (so random bytes outside NK aren’t part of the trust decision, at least not that gap)
•	Trailer CRC flip: immediate 00000006 (so gzip CRC is checked)
•	Payload change with correct gzip: 00000006 (so a second check exists)

ff
There must be a second check somewhere. Likely hash/crc of the decompressed bin stored somewhere. Possibly on internal flash... 


AIs new theory:
Yep, this lines up with what your SER_MSG.LOG is telling you, and it explains the behavior you’re seeing.

In the log, the bootloader is not “checking gzip”, it’s doing a separate container-level checksum over a big SD-card “App image” region that it reads by raw sector:
•	It reads 80 bytes of “program info” at sector 0x90000
•	Then it reads 73,963,728 bytes starting at sector 0x90001
•	Then it prints:

LoadSdCardImage_App : Check sum OK.
and earlier it shows:

2nd App seq:0x00000005, sum:0xFFFFCF9B, calced:0xFFFFCF9B -> 1

Also: the “first app seq” in the log has data that appears to be all zeros, and the calculated checksum comes out 0xFFFFFFFF, which strongly matches the algorithm you mentioned from swupdate.mef:

calced = bitwise NOT of (sum of all bytes, mod 2^32)

So when you change anything inside that 73,963,728-byte region (like your NK gzip member), gzip can be perfectly valid and you’ll still get 00000006 because the App image checksum in the 80-byte header no longer matches.

That’s also why:
•	flipping gzip header fields didn’t matter (container checksum still matched because you didn’t touch the big region in that test)
•	flipping gzip trailer bytes gave 00000006 (you changed the big region but did not update the App header checksum)

What you should do next

Use the known-booting image (update_civic_dump.img) as the reference to locate the checksum field(s) inside the 80-byte header at sector 0x90000, then patch those same field offsets in your modified image to the newly computed checksum.
REF:    update_civic_dump.img size=3963617280
TARGET: update_civic_dump.patched_crcok.img size=3963617280
App calced REF:    0x9CBCC015
App calced TARGET: 0x9CBCC015
Could not find REF checksum value inside the 0x80-byte App header.
Dumping header u32s for manual inspection:
+0x00: 0x046898D0
+0x04: 0xCF1EA698
+0x08: 0x00000000
+0x0C: 0x00000000
+0x10: 0x00000000
+0x14: 0x00000000
+0x18: 0x00000000
+0x1C: 0x00000000
+0x20: 0x00000000
+0x24: 0x00000000
+0x28: 0x622E4B4E
+0x2C: 0x007E6E69
+0x30: 0x00000000
+0x34: 0x00000000
+0x38: 0x00000000
+0x3C: 0x00000000
+0x40: 0x00000000
+0x44: 0x00000000
+0x48: 0x8E640000
+0x4C: 0x3B6806E0
+0x50: 0x00000000
+0x54: 0x00000000
+0x58: 0x00000000
+0x5C: 0x00000000
+0x60: 0x00000000
+0x64: 0x00000000
+0x68: 0x00000000
+0x6C: 0x00000000
+0x70: 0x00000000
+0x74: 0x00000000
+0x78: 0x00000000
+0x7C: 0x00000000


Wasnt what we wanted, So that “App header” and its checksum are for the big region starting around 0x12000200, not the NK gzip member at 0x24000000.
So, two layers are likely happening:
1.	App region validation (early blob at about 0x12000200 and friends)
2.	NK region validation (gzip CRC/ISIZE is one step, and then a deeper authenticity/integrity step over the decompressed NK image)




