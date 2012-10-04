---
date: 2011-02-15 20:54:50
layout: post
title: A Painful Squeeze
---

Tonight I finally got around to completing the upgrade of the server for this
site from Lenny to Squeeze. As upgrades go it was relatively painless. Well
painless in the same way that you can describe that having your arm ripped off
is painless when its then used as club to bludgeon you to death.

There was a time, as little as 10years ago (now that does scare the bejebus out
of me) when both my knowledge of operating a linux system was, well best
described as on a par with the knowledge that hedgehogs seem to have that roads
are a bad idea but they can't quite put their finger on why. Oh and the other
reason any kind of upgrade was difficult was because most of the software I had
to hand was shit.

Redhat 8, oh how I loathed you. Particularly the horrific memory leak you had on
my system. And the craptacular gnome theme which you.... oh fedora still uses
it....

Digression aside, the upgrade from Lenny to Squeeze went for the most part
without any hitches. The system remained up, and ssh access was maintained. The
web server restarted successfully and I was left with only two components which
no longer worked.

The first of those was the ftp daemon I run to make uploading and retrieving
multiple files easy. Strangely during the upgrade the package was lost which
meant a simple re-install and ftp was back online.

Then we have MySQL.


## MySQL

First up you need to remember that the version of mysql changed from 4.x to 5.x
between Lenny and Squeeze. This means that they fucked up the structure in any
existing tables you might be trying to migrate over.

At first I tried to run the upgrade scripts and do everything in what I
perceived to be the 'right' way.

But they didn't work. Or at least didn't work how the documentation described
they should.

So I gave up and did what I'd do again in the future, which is to purge the
database and start with a fresh install of mysql. Its then easy days to simply
take your database backup and pop it back into the server.


## The Moral

Is there a moral to this story and my very long winded rant? Well yes of course
there is or I wouldn't have made a header for it. Simply put, don't be afraid of
upgrades or doing things that you haven't before. The worst that can happen to
you is that you loose everything and have to start again from scratch.

But also hedge your bets and make backups of everything. I used **WP-DBManager
**to mail me periodic backups of my wordpress db which meant that I had the
security of knowing that if everything did go tits up that I'd probably be OK.

A\>
