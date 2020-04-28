---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "PineTime, OTA and Trust"
date: 2020-04-28T15:00:00+02:00
categories:
- PineTime
---

Recently there have been some discussion a OTA in the [PineTime Telegram group](https://t.me/pinetime).
This is my view on the trust model required for PineTime to work.

TLDR; I think we need to trust a new firmware to a certain extent before flashing
it to PineTime. A corrupt or malicious firmware can brick your device. With the
current hardware it will not be possible to design a system where you can recover
from all situations. Let me explain.

With laptops and smartphone you can recover from a corrupt or malicious
operating system by starting the BIOS or bootloader and select some kind
of recovery system. Then you can load a new or fixed OS using USB disk or
USB cable. I think we can't make such a system for PineTime. I think there
a few problem that make this impossible with the current hardware.

During firmware development this is not a problem, as you can use the SWD pins
inside the device to reload a firmware. In the processor is some kind of
mechanism (that is independent of the firmware) for writing to the flash and
resetting the processor. This means you can always recover from a incorrect
firmware.

The problems start when the back of the smartwatch is glued shut. This would
be the case for production watches (not development focused). And some of the
community members already have glued their development watches. With the back
closed you can't reach the SWD pins and have to upload a new firmware in a
different way.

The PineTime has Bluetooth Low Energy available, which is the perfect candidate
for firmware upgrade. See a previous post about [Over-The-Air updates](../mynewt-ota/).
This will work perfectly for correct functioning firmwares.

The OTA mechanism that mynewt uses will write the new image to flash
separate from the running image. During next boot of the device the bootloader
will swap the two images and start the new image. This way you can fallback
to the previous image when a problem occurs.

The difficulty is to determine when a problem occurs. It will be up to the
firmware to report that there is a problem. But the firmware is also the thing you
are unsure about. The user has three main ways to communicate with the device:
the touch screen, BLE and the hardware button. The are all under the control of
the firmware (the touch screen needs some kind of user interface; BLE needs a host;
the button needs to be handled in software).

A solution would be to restart into the bootloader, so that you can revert or
upload a new firmware. But how do you do that? All methods of communication with
the device are under control of the firmware, which could be corrupt.
The only reliable way to reset the
device is to fully drain the battery, which anecdotally could take weeks.

As I see it you need to trust that the new firmware is able to revert or reload
a different firmware onto the device. If you don't trust the new firmware, you
should not upload it to your PineTime.

I do have some ideas about building some kind of service to automatically
test this recovery behavior of a firmware. This could give you the confidence to try a
new image or integrate into CI to make sure you don't release a corrupt firmware
to your users. But more on that at a later time.
