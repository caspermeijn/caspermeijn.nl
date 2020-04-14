---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "Mynewt over the air update"
date: 2020-04-14T12:00:00+02:00
categories:
- PineTime
- mynewt
---

I have done a little experiment with Over The Air updates for PineTime firmware. I
used the mynewt project from my [last post](../pinetime-mynewt/) as a base. In a few hour
I had updated my firmware.

I started with the blinky project from the last post. I expanded that to the
[BLE Peripheral App](https://mynewt.apache.org/latest/tutorials/ble/bleprph/bleprph-sections/bleprph-app.html).
This is a basic application that presents itself as a Bluetooth service. When it runs
you can connect to it from your smartphone.

The cool thing is that you can also connect to it using [newtmgr](https://mynewt.apache.org/latest/os/modules/devmgmt/newtmgr.html)
which is an protocol for device management. The bleprph firmware has most of the features enables.
You could get some stats, set the time and read some long term logs. Lots to play with ;-)

I was primarily interested in the [OTA upgrade functionality](https://mynewt.apache.org/latest/tutorials/devmgmt/ota_upgrade_nrf52.html).
I first installed the [newtmgr](https://github.com/apache/mynewt-newtmgr) application on my laptop. I already
had go-lang installed, so it was relatively easy. Then I could do `newtmgr --conntype ble --connstring peer_name=nimble-bleprph image list`
to show the current image state.

Then you can upload an new firmware image `newtmgr --conntype ble --connstring peer_name=nimble-bleprph image upload /home/casper/src/mynewt-pinetime-demo/bin/targets/bleprph-pinetime/app/apps/bleprph/bleprph.img`,
which takes a long minute. The `list` command will now show two versions.

Now you can select the new image using `newtmgr --conntype ble --connstring peer_name=nimble-bleprph image test`.
After rebooting the device (I have not yet found a good way to do this, for now I short-circuit
the 3.3v to ground, but that is not a best practice), it will run the new image. Cool! 
You just need to confirm that the image works correctly using `newtmgr --conntype ble --connstring peer_name=nimble-bleprph image confirm`,
else it will load the old image during next reboot.

See the [mynewt Over-the-Air Image Upgrade tutorial](https://mynewt.apache.org/latest/tutorials/devmgmt/ota_upgrade_nrf52.html)
for better instructions.
