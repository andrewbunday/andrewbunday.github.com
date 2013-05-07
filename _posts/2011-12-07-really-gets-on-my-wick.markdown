---
date: 2011-12-07 18:33:52
layout: post
title: "Maya <small>Plugins without out shelves really get on my wick</small>"
tags:
- mel
- maya
---

This kinda thing really gets on my wick. Sometimes a plugin comes along which
doesn't source it's own shelves. When that happens you have to hunt around in
the file system to load the shelf. So instead popping this file into the scripts
directory of the plugin

{% highlight python %}
import maya

def bbLoadShelf(name, filepath=""):
    mainShelfLayout = maya.mel.eval('global string $gShelfTopLevel; string $tmp=$gShelfTopLevel;')
    labels = maya.cmds.tabLayout(mainShelfLayout, query=True, tabLabel=True)
    if not name in labels:
        maya.mel.eval('addNewShelfTab "%s"' % name)

maya.utils.executeDeferred( bbLoadShelf('MyPlugin')
{% endhighlight %}

