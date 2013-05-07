---
date: 2012-10-16 10:00AM
layout: post
title: "Puppet <small>One time PPA additions</small>"
synopsis: "Puppet exec has a 'creates => somefile' action. Use it to ensure an action only occurs when it needs to."
thumb:
tags:
- puppet
- apt
- ppa
---

### Contents
{:.no_toc}

---

* This will become a table of contents (this text will be scraped).
{:toc class="nav nav-list well"}

## Background
---

When running puppet on our hosts we were finding that it would take not only far longer than we expected it to but that it was re-applying the creation of PPAs each time. These PPAs are critical for keeping our software up to date so we needed to find a way to keep them from being re-created on each run.

## Apt
---

At Baseblack all of our workstations, render nodes and servers have their software packages, configured and managed via a combination of Puppet and Apt repositories.

We preseed the build of a new host, pointing the installer at the official Ubuntu mirrors to retrieve the packages it requires. There are however many optional packages not yet included as part of universe or multiverse which our users find useful for their day to day work which we include on the system.

For internal software and tools we use Jordan Sissel's [fpm](https://github.com/jordansissel/fpm) to generate Debian packages which can then locally host in a [reprepro](http://mirrorer.alioth.debian.org/) managed apt repository.

For those external pieces of software which are not in the official repositories we heavily rely upon Ubuntu's [PPA](https://help.launchpad.net/Packaging/PPA) mechanism for adding additional unofficial packages to the system.

## PPAs
---

PPAs were developed on top of Launchpad as a means of hosting and distributing software. Some examples of the software which we install using PPAs are; salt for orchestrated tasks, hugin the panorama stitcher and the latest nvidia graphics driver.

The easiest way to add a ppa to the sources list is to use the add-apt-repository command. This takes the ppa's short name, builds the apt url to access it, stores it in `/etc/apt/sources.list.d/*` and retrieves its public signing key.

> add-apt-repository ppa:username/package

## Puppet
---

We make use of the puppet-apt manifests put together by evolvingweb.

> https://github.com/evolvingweb/puppet-apt

Unfortunately each time puppet was running we were finding that it would not only take longer than we expected, but that the time it took was variable. Often during the middle of the day when our internet connection was more heavily being utilised the runs would take substantially longer.

We quickly saw when inspecting the run logs that on each run the ppas were being re-added into *sources.list.d*.

Using what little skill I have at writing puppet configurations and from **_reading_** the documentation I amended the apt::ppa definition to make use of the **creates =>** parameter for the **exec** action. This parameter causes the action to only be executed if the file which it points towards does not exist at the specified location.

After adding the condition we found that our runs became much shorter and consistent in run time.

Finally here is the amended definition in all of its terrible glory:

{% highlight puppet %}
    define apt::ppa() {
        require apt

        $ppa_prefix = split($name, ':')
        $ppa_name_repo = split($ppa_prefix[1], '/')
        $ppa_name = $ppa_name_repo[0]
        $ppa_repo = $ppa_name_repo[1]

        exec { "apt-update-${name}":
            command     => "/usr/bin/aptitude update",
            refreshonly => true,
            creates => "/etc/apt/sources.list.d/${ppa_name}-${ppa_repo}-${lsbdistcodename}.list",
        }

        exec { "add-apt-repository-${name}":
            command => "/usr/bin/add-apt-repository ${name}",
            notify  => Exec["apt-update-${name}"],
        }
    }
{% endhighlight %}

## TL;DR
---

Puppet exec has a 'creates => somefile' action. Add it to the exec definition which calls add-apt-repository to make it only run the first time before the file has been created.
