---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "Read It Later 0.6.0"
date: 2024-07-11T00:00:00Z
categories:
- Read It Later
---

A new release of Read It Later is now available on [Flathub](https://flathub.org/apps/details/com.belmoussaoui.ReadItLater). This update features a redesigned "new article" user interface and faster startup times for accounts with numerous articles.

<!--more-->

Since the last release of [Read It Later 0.5.0]({{< ref "/posts/read-it-later-0.5.0.md" >}}), I have been working on improving the app's internal structure. This includes updating the application metadata, setting the correct minimum window width, and fixing deprecation warnings.

#### Redesigned "Add Article" Interface

The "add article" user interface has been completely redesigned. This change was prompted by internal restructuring, specifically moving the header bar to be a part of each page. In the old design, the input field replaced the contents of the header bar:

![Old design of "add article" user interface](/read-it-later-0.6.0-add-article-old.png)

It was challenging to make this work with the header bar as part of the article list page. Therefore, I decided to open a separate window for entering the article URL. The URL in the input field is now validated, and if a URL is on the clipboard, it is automatically pasted when the window opens. By using [AdwDialog](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/class.Dialog.html), the window is automatically centered and scales well on mobile devices.

![New design of "add article" user interface](/read-it-later-0.6.0-add-article-new.png)

#### Performance Improvements

My personal Wallabag account contains more than 2,500 articles, which caused Read It Later to load very slowly. I used profiling to identify the pain points. While logging into the account is still somewhat slow, it is now more manageable. Here are some of the improvements I've made:
- **Sorting articles using [SortListModel](https://docs.gtk.org/gtk4/class.SortListModel.html):** Previously, the article list was kept in a sorted state. Adding an article involved scanning all existing articles to find the correct position, then moving all elements in the list to make space for the new article. SortListModel, being a proxy, only sorts the references to the articles, which is much more efficient.
- **Adding loaded articles in one batch:** Since the list of articles no longer needs to be sorted, I can add all loaded articles in one batch. Previously, articles were added one by one, with the UI updating in between each addition. Now, the UI updates only once.
- **Using [ListView](https://docs.gtk.org/gtk4/class.ListView.html) for rendering the article list:** ListView is smarter and only renders the articles that are actually visible. As you scroll, the next batch of articles is drawn. This is much more efficient than rendering all articles at once.
- **Caching the preview text in a database:** The first paragraph of each article is shown as a preview in the article list. This text was previously generated by parsing the full article HTML every time the application started, which was very resource-intensive. Now, the preview text is generated once for each article and saved in the article database.

#### Other Improvements

I fixed an interesting bug: pressing the ESC key on the login page would transition you to the article list, but since you were not logged in, the article list was empty. This transition was intended for the article view page but was enabled for all pages. A simple check fixed the issue.

Finally, I updated the app’s dependencies. Read It Later uses many libraries, and I updated the runtime to GNOME 46. This includes all the libraries developed by the GNOME community, such as GTK, webkit2gtk, and libadwaita. The new library versions recommend some code changes to follow best practices, which generally results in better code.

Thanks to the [translators](https://l10n.gnome.org/module/read-it-later/) for their continuous contributions.

<a href='https://flathub.org/apps/details/com.belmoussaoui.ReadItLater'><img width='240' alt='Download on Flathub' src='https://flathub.org/assets/badges/flathub-badge-en.png'/></a>