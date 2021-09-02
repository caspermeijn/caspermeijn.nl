---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "HP Proliant Gen8: Boot from USB disk"
date: 2021-09-02T14:38:06+02:00
categories:
- HP Proliant Gen8
- Fedora
---

Recently I bought a secondhand HP Proliant Microserver Gen8 G1610T. It included a SSD drive in slot 1 and two HDD drives in slots 3 and 4. I want to add two more HDD drives in slot 1 and 2. Therefore the SSD had to move to the ODD slot. In this model there is no CD-rom drive installed, so that space is available. 

I had installed Fedora Server on the SSD, moved the SSD to the ODD slot and the server didn't boot anymore. As can be found on many places on the internet, the BIOS will always boot from the first installed disk and I move the SSD to the last slot.

Luckily the internet could tell me that an USB disk is preferred over the other disks. As I didn't want to use a slow USB drive as boot disk, I wanted to create a USB drive that uses GRUB to chainload the SSD. So on my laptop I installed GRUB to the USB drive, added configuration and placed the drive in the server. Reboot the server and it didn't work. After multiple hours of tinkering I had not been able to make it work. 

Eventually I gave up and moved the SSD back to slot 1. I wanted to try a last option: run `grub-install` from a booted Fedora. I rebooted again and it worked flawlessly. 

To make sure no boot files were installed to the USB drive I wipe the drive and intalled GRUB again. The only thing installed to the USB is GRUB, the rest is loaded from the SSD. This is exactly what I wanted.

```shell
[casper@novac ~]$ sudo parted /dev/sdd
GNU Parted 3.4
Apparaat /dev/sdd wordt gebruikt.
Welkom bij GNU Parted!  Typ 'help' voor een opdrachtenoverzicht.
(parted) mklabel msdos                                               
Waarschuwing: Het bestaande label op /dev/sdd zal worden vernietigd en alle gegevens op deze schijf zullen verloren gaan.  Wilt u doorgaan?
Ja/Yes/Nee/No? yes                                                        
(parted) mkpart primair fat32 512M 1024M
(parted) zet 1 opstart                                                    
Nieuwe toestand?  [aan]/on/uit/off?                                       
(parted) einde                                                            
Opmerking: Het kan nodig zijn /etc/fstab bij te werken.

[casper@novac ~]$ sudo grub2-install /dev/sdd                                    
Installeren voor i386-pc-platform.
Installatie is afgerond. Er werden geen fouten gerapporteerd.
[casper@novac ~]$ sudo reboot
```
