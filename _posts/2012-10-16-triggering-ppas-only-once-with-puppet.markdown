---
date: 2012-10-15 10:00AM
layout: post
title: "Triggering PPAs only once with Puppet"
synopsis: "Puppet exec has a 'creates => somefile' action. Use it."
---

At Baseblack all of our workstations and rendernodes are configured and managed via a combination of Puppet and Apt repositories.

We use Ubuntu's offical mirrors for most of the base OS install and from their install many of the optional packages which our users find useful in their day to day work.

Whilst our internal software and tools we add onto the base distribution are packaged using [fpm](https://github.com/jordansissel/fpm) and store them in a locally hosted apt repository using [reprepro](http://mirrorer.alioth.debian.org/).

However there are always a few packages which slip inbetween the two. Software which is being actively maintianed and updated, but which hasn't yet made it into the offical distribution.

For these we heavily rely upon Ubuntu's PPA mechanism for adding additional unofficial packages to the system.

## PPAs

PPAs were developed ontop of the Launchpad service (which personally i've never been able to get along well with)

what has caused us an issue and what feature of puppet has caused this

what feature of puppet can we use to solve this
