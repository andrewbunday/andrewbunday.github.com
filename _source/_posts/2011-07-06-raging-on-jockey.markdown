---
date: 2011-07-06 17:16:15
layout: post
title: Raging on Jockey
---

Recently I've been raging out quite frequently against some of the technology
I've been having to come acquainted  with. In particular there is Puppet a
centralised configuration tools which we've been using to orchestrate our
automated deployment system in conjunction with apt-get and reprepro as our
backend package respository here at Baseblack.

However describing the system we've been building is the topic of another for
longer and more interesting post. Where I can cover some of the hurdles we've
encountered and how we've dealt with them, despite some of the frankly bizarre
behaviours which Puppet exhibits.


### Bloatware

No, instead I'm going to pick on **jockey-text**. Jockey-text is the command
script version of the gtk interface provided with Ubuntu for the last few
releases for installing hardware drivers attached to the system. Specifically
graphics card adaptors, printers and firmware updates are handled by jockey. To
cope with such a broad range of packages to install, kernel modules to enable
and devices to probe for Jockey has become something of a behemoth.

The jockey-text executable itself is quite short, mearly wrapping the jockey
python module. However the module's source runs at close to 4 1/2K lines of
code. This 4500 lines is taken up by numerous; private nested functions, class
methods and dbus-functions which bloat Jockey massively beyond what should be
required of it.


### Exit Codes

However none of this should have mattered to us. Instead jockey should be doing
its thing and leaving us to get on with what we've been working on. To get a
workstation net installed and completely configured without human interation.

Unfortunately despite its near 4.5K of code, there is very little good exception
handling or adherence with the rules of program exit codes. You see should it
not get to the end of its run, because for example the driver your asking to be
enabled, has already been. Then jockey will immediately exit with a value of 1.

That makes it pretty darned hard to use with any confidence.


### Installing a graphics driver

In our the action we wanted jockey to do was enable the nvidia driver which
we're apt-getting from the x-swat ubuntu ppa. Getting annoyed with jockey, and
not particularly wanting to have to fight with Puppet, I decided to have a dive
into the jockey source and see what its really doing.

Here I present an immensely cut down version of what it's really doing (in our
edge case).

Pretty simple huh? I'm still undecided about how we're going to go about
installing our nvidia drivers. Although I have the sneaking susupission that
we'll probably be _apt-get installing_ them ourselves and then calling _nvidia-
xconfig_ to enable them before rebooting at the end of the installation.
