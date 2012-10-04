---
date: 2011-07-06 18:49:50
layout: post
title: "Update: Raging on Jockey"
---

After posting up a short rage post about [Jockey](http://www.andrewbunday.com/2011/07/raging-on-jockey/)
I decided to dive into the deb package file which nvidia current is installed
from and I realised that the postinst script actually does most of the heavy
lifting for me when installing the driver.

It first runs **update-alternatives** on a modprobe.conf file to ensure that the
nouveau driver is blacklisted. Next is adds the kernel module into **dkms** and
starts the autoinstaller. Finally it runs **modprobe** to add the kernel module
into the kernel.

Which made me realise that jockey brings absolutely nothing to the table. Its
now been replaced with these two lines of effective shell script:

    $ apt-get install nvidia-current
    $ nvidia-xconfig

Awesome.
