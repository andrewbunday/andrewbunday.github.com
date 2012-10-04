---
date: 2011-12-07 19:02:18
layout: post
title: Greenlet based PyQt and Flask Application
tags:
- code
- python
---

I had a slightly odd moment a few weeks ago and felt that I really didn't want to write a PyQt based desktop application.
Instead, fueled by some webapp based docs i'd been reading recently (I blame you @manneohrstrom) which were written using Flask. I decided to try out launching a PyQt QWebView and pointing it back at a local webservice run from the same process.

What I great idea I thought! Jumping in blindly only to quickly discover that both Flask and PyQt want to run in the primary python thread. To get around the problem I fell back onto Greenlets and hand processing events in PyQt rather than giving over control to app.\_exec\(\).

Ultimately I ended up ditching the whole thing and reverting back to a vanilla
PyQt application since once I started to add user interaction for elements such
as dynamic combo boxes I found that my Javascript skills were sufficiently rusty
that it was taking an age to write the handlers, and callbacks on both ends of
the system.

<script src="https://gist.github.com/1358794.js"> </script>
