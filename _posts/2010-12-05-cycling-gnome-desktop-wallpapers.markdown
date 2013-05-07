---
date: 2010-12-05 00:14:43
layout: post
title: Gnome <small>Cycling Desktop Wallpapers</small>
tags:
- gnome
---

If you are like me then you get bored fairly easily. Something which in today's
constant dip information world is all to easy. An example for me is finding that
I like to have an attractive background on my PC but get bored looking at the
same one for any period of time.

On most recently released systems it's fairly easy to go to the default desktop
background manager and select a series of images to cycle through along with a
dwell period for each to be displayed.

Apparently that isn't the case for Gnome.

If you go to the background manager you can select single images, and, crucially
select a pre-defined list of images to cycle through. But there is not facility
to do this yourself. More accurately there isn't any GUI to help.

It turns out an xml file is needed to be stored under:

    /usr/share/gnome-background-properties/

In this file you name another xml file which defines the images to display and
the display interval for each.

Here are the two files I use:

[darkwallpapers.xml](http://sinotr.eu/darkwallpapers/darkwallpapers.xml)
[background-1.xml](http://sinotr.eu/darkwallpapers/background-1.xml)

So, for example we end up with:

    /usr/share/gnome-background-properties/darkwallpapers.xml
    /home/andrewb/Pictures/darkwallpapers/background-1.xml

Here is a link to a zip including the files and pictures which I use:
[darkwallpapers.tar.gz](http://sinotr.eu/darkwallpapers/darkwallpapers.tar.gz)
