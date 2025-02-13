---
layout: post
title:  "New Car and Reverse Engineering It"
comments: true
date:   2025-02-12 14:55:00 -0500
categories: honda civic reverse engineering hack hacking car auto automotive
---

<!-- Content -->

I recently had the pleasure of acquiring a dream car of mine. The car is a manual. It comes in a sporty blue color, has a nice but subtle wing on the back. It has a nice black interior with red lined seats and carbon fiber accents on the dash.

If you’d have guessed a Lamborghini or some other similar car, you would be very wrong. No, the car I’ve purchased is in fact a 2014 Honda Civic Si. That’s right, the last Civic with true VTEC. 

![2014 Honda Civic Si](https://hips.hearstapps.com/hmg-prod/amv-prod-cad-assets/images/14q2/584476/2014-honda-civic-si-sedan-test-review-car-and-driver-photo-597226-s-original.jpg "2014 Honda Civic Si")
_Photo credit: Car and Driver_

Now, there isn't really anything about this car that I don't love. It's super practical, allowing me to haul around my wife and our son in his car seat, along with a stroller, groceries, two other people, and probably even more. It gets pretty good gas mileage (when I'm not totally mashing the throttle to experience that sweet VTEC), and cost of ownership is very low. The interior is very comfortable, and looks modern, for a now 11-year-old car. 

But, there is one thing that has haunted this car and other Honda models from this 2014-2016 (and even some later years too): lack of Apple CarPlay and Android Auto.

You see, Honda promised many customers that these vehicles would eventually receive updates to enable this functionality. Hell, the [Apple CarPlay website from early 2015](http://web.archive.org/web/20150311070813/https://www.apple.com/ios/CarPlay/) literally has a picture of a 9th gen. Honda Civic
interior in it running CarPlay. **bruh moment**. Of course, many purchasers were rightfully frustrated because their headunit had been abandoned, never receiving a single software update. 

This lack of modern infotainment functionality really sucks. The 7 inch display in these vehicles is gorgeous in my opinion, and the center stack just looks amazing. Even the existing software on the unit is decent, and the size of the screen would handle CarPlay and Android Auto just fine. 

OK, so now you've heard my complaint. After much research, the general consensus is you have three options (currently):

### Option 1

Your first option is to do the swap everyone is talking about: take the radio from a 2016 Honda CR-V, install it in the Civic, then update the radio software on the new unit. This is a good option
if you want to maintain the factory look and feel, as well as other software options like trip computer and vehicle settings. This option costs anywhere between $200-$300 USD, depending on where you source 
the radio, bracket, and faceplate (if needed). Even though this is a good solution, the CarPlay update was completely beta software, and a lot of people report stability issues. Plus, I personally do not
like the look of the CRV radio as much with the big chunky buttons on it. Finally, no Android Auto with this one.

### Option 2

Your second option is going to Crutchfield and purchasing whatever double DIN unit you want, plus all of the adapters needed to remain Lane Watch, factory controls, and more. This option is quite a bit pricier,
in my case for the unit I looked at and all cables (iDataLink) around $500, not including the face plate I would need to make it work since the stock unit is not a double DIN size. This option
is probably the best for the most modern functionality, and Crutchfield is a great brand that would be able to guide through any issues you run into. A new unit also likely means better sound output and EQ settings.

### Option 3

Similar to option two, the third option is also an aftermarket radio, but more specifically the Joying unit advertised specifically for the 2014-2015 Honda Civic. This unit can potentially
keep all factory functions (excluding trip meter control, as far as I know) such as Lane Watch, steering wheel, etc. I have heard from a handful of folks that have this unit that its a good option,
especially with the large 9 inch screen. However, this unit is still almost $500, and the carbon fiber trim it comes with is different than the current OEM carbon fiber trim. I'm not sure about all of you,
but that might just bug the crap out of me every time I look at it. 

## Why not go with one of the current options?

Well, I think the presented options are all generally pretty good. Option 2 is by far my favorite, and the one I would go with if I picked from one of them. But I still think the factory unit looks great,
and would really like to see CarPlay somehow running on it. Where does that lead me?

As a Software Engineer and a tinkerer, I like to figure out how things work, and see if I can make them work how I want. With that in mind, I've set out to see if I can reverse engineer the factory
Honda 7 inch headunit to add Apple CarPlay, Android Auto, and potentially other functionality too, like connecting with an OBD device and displaying real time gauges. 

## Where to start?

I began doing a ton of research. Figuring out what the unit runs, what cars run it, see if anyone else has tried this, everything. I also ended up purchasing a unit of eBay so that I can tear it down
and mess with it so I don't mess up the one currently installed in my car. 

Here is some info I learned:
- The unit runs Windows CE 7.0
- The unit is manufactured by Mitsubishi Electric
- It has a Renesas Electronics R8A77780 chip as its main processor
- It has Micron Technology D9PTJ NAND chips for flash memory
- There are lots of hidden diagnostic and factory menus (which I plan on mapping and exploring more)
- No obvious serial or JTAG port that I can identify currently

I have the radio I purchased powered up on a bench setup. I tore it down to take internal pictures, but need to get better ones. For now, I am poking around the software and am attempting to find a way 
into the Windows CE environment.

## How would CarPlay/AA run?

I have a few possible solutions to this.

### Solution 1

Every Honda equipped with this headunit has an HDMI port. Honda originally sold an adapter that connected to iPhone devices only via lightning that connected to the HDMI port and USB port on the vehicle.
Then, you could download the HondaLink app, and purchase the navigation app as well. If you plugged in using this setup, and connected the phone via Bluetooth as well, you could run navigation and other apps
on the headunit, powered by the phone itself. This worked while driving too, even though other HDMI display is not allowed while driving (probably some special HondaLink app thing).

My theory is that the HDMI was used for
video signals, the USB was used to send touch data from the headunit screen back to the device, and the bluetooth was for audio. If that theory is true, then we could possibly exploit this and "mimic" the behavior
using a Raspberry Pi. If this could happen, then the Raspberry Pi could run CarPlay or AA on it, and anything else one might want. This would require understanding the setup completely though, and ensuring
that you can do the same handshake the HondaLink app was doing to allow video display while driving. Unfortunately this version of the HondaLink app is no more, so reverse engineering the app itself is not possible. 

### Solution 2

Since the unit runs Windows CE, it would theoretically be possible to gain access to the actual desktop environment. If you could, it may be possible to get a better understanding of the handshake process. Hell, the .exe files themselves
could even be reverse engineering in most cases to see what steps they take to allow Solution 1 to work. It may be possible to load some custom software on the unit, even a new .exe with CarPlay or something. But this solution is less viable than 
solution 1. However, gaining access to the .exe files would still be ideal for both solutions.

## Next steps

Now that I have begun down this road, I can't stop. So far there have not been any quick and easy wins like I was hoping. I initially was hoping for a JTAG, UART or serial port on board internally, but do not see one
(at least not an obvious one). I'm going to keep digging though, and in my next post I hope to have lots of pictures and potentially video walking through the process.

As with any project like this, community support is greatly appreciated. Honda essentially abandoned all owners with this headunit, and I would like to see to it that we can in some way get CarPlay/AA on the factory 7 inch unit. 
I would love for others that have any information on this unit or any knowledge that may help this project along to chime in and assist. It would be **greatly** appreciated. 

Peace out!