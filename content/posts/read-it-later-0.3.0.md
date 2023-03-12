---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2023 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "Read It Later 0.3.0"
date: 2023-03-11T08:26:32Z
categories:
- Read It Later
---

A new release of Read It Later is available on [Flathub](https://flathub.org/apps/details/com.belmoussaoui.ReadItLater). The significant changes are the upgrade to GTK 4, some fixed bugs, and new translations.

<!--more-->

I have been using [Wallabag](https://www.wallabag.it/en) for a year as my read-it-later application. Whenever I find an interesting web article, I add it to my Wallabag account. Then when I have a few minutes to spare, I open the Wallabag app on my Android phone to read them. I love the workflow and don't have to doomscroll any social media.

Meanwhile, I have an interest in Linux on mobile phones. I own a [Librem 5](https://puri.sm/products/librem-5/) and a [PineTab](https://www.pine64.org/pinetab/), which run mobile Linux with Phosh. In the app store, I found a great app called [Read It Later](https://flathub.org/apps/details/com.belmoussaoui.ReadItLater), which allows me to read my articles from my Wallabag account.

A few months after discovering the app, it started complaining about an old runtime version. As a good open-source citizen, I wanted to report this issue. In that process, I found a similar issue: [Port to GTK 4](https://gitlab.gnome.org/World/read-it-later/-/issues/58). The creator of the app, [Bilal Elmoussaoui](https://belmoussaoui.com/), is looking for someone to do this work. Then I saw that the app is written in [Rust](https://www.rust-lang.org/), my favorite programming language! I found a new hobby project 😎

I followed the migration guides for [GTK 4](https://docs.gtk.org/gtk4/migrating-3to4.html) and [libadwaita](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/migrating-libhandy-1-4-to-libadwaita.html). I implemented some of the best practices found in the [gtk-rs book](https://gtk-rs.org/gtk4-rs/stable/latest/book/). I fixed some [bugs](https://gitlab.gnome.org/World/read-it-later/-/issues/57) along the way. And (where it all started) I updated to the latest [GNOME runtime](https://developer.gnome.org/documentation/introduction/flatpak.html).

I really like the tech stack for this application. GTK and libadwaita provide great UI widgets. Rust provides high-quality libraries and a lot of compile-time checks. [gtk-rs](https://gtk-rs.org/) provides bindings from Rust to GTK. Flatpak provides a stable build and runtime environment. The [VSCode Flatpak plugin](https://marketplace.visualstudio.com/items?itemName=bilelmoussaoui.flatpak-vscode) provides the editor integration.

Thanks to Bilal for mentoring me and thanks to the [translators](https://l10n.gnome.org/module/read-it-later/) for the constant stream of contributions.

<a href='https://flathub.org/apps/details/com.belmoussaoui.ReadItLater'><img width='240' alt='Download on Flathub' src='https://flathub.org/assets/badges/flathub-badge-en.png'/></a>
