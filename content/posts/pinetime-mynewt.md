---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "PineTime Mynewt"
date: 2020-04-04T20:14:15+02:00
categories:
- PineTime
---

So I recently bought a [PineTime dev kit](https://store.pine64.org/?product=pinetime-dev-kit).
It is a development kit for a smartwatch. It is complete, but not fully assembled. The back is
loose, so that you can program your own software. Pine64 is a hardware company, which expects
the community to write open-source software for it. 

mynewt
------

After seeing some great work of [lupyuen](https://github.com/lupyuen/pinetime-rust-mynewt) I
got inspired to do some of the ground work to make the PineTime upstream supported by the
[mynewt operating system](https://mynewt.apache.org/). It is a small operating system that is
focused on IOT type devices and it has a powerful Bluetooth Low Energy stack. It appears to
focus on good practices, like security and management standards.

So I started to work on a [PineTime Board Support Package](https://github.com/apache/mynewt-core/tree/master/hw/bsp/pinetime).
This is the configuration of the OS, so that it knows which hardware it will run on.
And is contains instructions for the `newt` tool for programming the device.

It is now accepted upstream, so that means it can be used more widely. However there are some
gotchas you need to be aware of, that is the reason for this post.

Tutorial
--------

I am very pleased with the [documentation of mynewt](https://mynewt.apache.org/latest/). I was
able to start making software in no time. So I contributed a tutorial of my own. Currently the 
merge request has not yet been merged, but you can view it in on [Github](https://github.com/apache/mynewt-documentation/blob/5642718c5af9ae09eab47feb6eb3688040c70ffa/docs/tutorials/blinky/pinetime.rst).

The turorial will guide you to a blinking backlight on the device. First you need to do need to setup the mynewt
toolchain using [this page](https://mynewt.apache.org/latest/get_started/index.html). The you need to
read up on [project blinky here](https://mynewt.apache.org/latest/tutorials/blinky/blinky.html). Then you can
start the [tutorial here](https://github.com/apache/mynewt-documentation/blob/5642718c5af9ae09eab47feb6eb3688040c70ffa/docs/tutorials/blinky/pinetime.rst).

I have two additional note:

- I had to use the upstream OpenOCD to make this work. I build the master branch of [this repository](https://github.com/ntfreak/openocd).

- Before doing `newt upgrade` you need to select the master branch of mynewt, as the PineTime BSP is not 
    released yet. You need to open project.yml and set `vers` to `0-dev` for the repository `apache-mynewt-core`.
    Then do the `newt upgrade` and you will see that the 0.0.0 master branch version is selected.

For the rest the tutorial will guide you to a working application.

Drivers
-------

Then a note on the drivers for the PineTime hardware. The current BSP supports the System-On-a-Chip
that is used in the PineTime. It also supports the Nimble BLE stack. But most of the other
chips on the device are not supported. I plan to slowly add some drivers as I need them.
