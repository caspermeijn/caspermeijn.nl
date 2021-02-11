---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "EdgeRouter SFP X"
date: 2021-02-09T11:30:45+01:00
---

These are some instructions to connect my Ubiquiti EdgeRouter SFP X to my internet provider XS4all.
This post is primarily documentation for myself, but they may be helpful to you.

# Wizard Basic Setup
- Internet port: `eth5`
- Connection type: `PPPoE` with username `xs4all` and password `xs4all`
- Choose `Internet connection is on VLAN` with ID `6`
- Choose `Enable DHCPv6 Prefix Delegation` with prefix length `/48`
- Uncheck `Only use one LAN`
- Setup new admin user

# Configuration
After reboot:
- On Dashboard, click `Actions` for eth5 and choose `Config`
- Change `Speed/Duplex` to `1000/full`
- Click save
    
Then some settings that are unavailable via the normal web interface. This can be done via the Config Tree or the CLI:
- `set interfaces ethernet eth5 vif 6 pppoe 0 ipv6 enable`
- `set interfaces ethernet eth5 vif 6 pppoe 0 ipv6 address autoconf`
- `set interfaces ethernet eth5 vif 6 pppoe 0 dhcpv6-pd prefix-only`
