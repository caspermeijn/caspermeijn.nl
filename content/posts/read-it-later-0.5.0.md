---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "Read It Later 0.5.0"
date: 2023-10-30T10:21:44Z
categories:
- Read It Later
---

A new release of Read It Later is available on [Flathub](https://flathub.org/apps/details/com.belmoussaoui.ReadItLater). The highlights are a crash fix and better support for links in articles.

<!--more-->

My last post was about [Read It Later 0.3.0]({{< ref "/posts/read-it-later-0.3.0.md" >}}). I have continued improving the internal structure of the app, which includes improvements to AppData (which is content shown in Linux App Stores), making sure the correct texts have translatable markings, and modernizing the internal code structures to best practices.

I have fixed article synchronization for some users with an older account. Those accounts had imported articles with an older version of [Wallabag](https://www.wallabag.it/en), which allowed a NULL value as the content type. Read It Later didn't expect a NULL value and crashed. With the new version, a NULL value is simply ignored, and the sync works as expected.

A bug that slipped in that caused a crash when right-clicking an article. A new version of the WebView meant a changed function signature. The compiler doesn't check this function signature, and I did not check manually (by right-clicking the WebView). The fix was to change the function signature.

The right-click on the article is captured to turn off options that are not useful, such as navigation back and forth. In version 0.5.0, I disabled additional options, such as downloading links and images and opening links in a new window. While at it, I implemented opening links in the browser instead of inside Read It Later.

The last category of changes is regarding dependencies. Read It Later uses a lot of libraries. I updated the runtime to GNOME 45; this includes all the libraries developed by the GNOME community, such as GTK, webkit2gtk, and libadwaita. The new library versions recommend some code changes to follow best practices. This causes some churn but generally results in better code.

Thanks to the [translators](https://l10n.gnome.org/module/read-it-later/) for the constant stream of contributions.

<a href='https://flathub.org/apps/details/com.belmoussaoui.ReadItLater'><img width='240' alt='Download on Flathub' src='https://flathub.org/assets/badges/flathub-badge-en.png'/></a>
