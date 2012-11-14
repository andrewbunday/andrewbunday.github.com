---
date: 2012-11-12 18:05PM
layout: post
published: true
title: "ReproWeb <small>a micro repository browser</small>"
synopsis: "Reproweb is designed to provide insight into your self hosted apt repository. It is built on top of the standard debian dpkg toolchain and the repository builder/mirroror reprepro."
---

* This will become a table of contents (this text will be scraped).
{:toc}

---

## Overview

A few weeks ago we finished a first draft release of [ReproWeb](http://github.com/andrewbunday/reproweb), a small, light weight apt repository browser. The browser was build out of an increasing need we were finding, to be able to introspect into our main production repository.

Over the course of 1.5 years that repository has grown to the point when the main stable channel contains around 334 individual packages whilst the unstable channel contains a further 149.

Without running apt-cache commands, and munging with our software source lists it was becoming difficult to not only find the name/version of a piece of software, but it was also becoming hard to tell:

* When the package was created
* Who created the package
* If the package had a newer version in the unstable channel
* If the package was available in the stable channel
* If the package had been published for all of our supported distributions.

Almost all of these tasks can be carried out from a terminal using commands such as `apt-get show`, but the web interface allows for far quicker navigation, comparison and has already started us asking more questions as our understanding of the data in our repository grows.

---

## Current Feature Set

At present Reproweb allows a user to perform filters on the content of a repository based on either;

* simple string pattern matching against either the package name, version string or codename/component/architecture,
* or by browsing through the hierarchy of the repository from:

    ```
    codename -> component -> architecture -> package -> version
    ```

### Browsing

> Do you find yourself wanting to know more about your repository? Perhaps how many packages are in a particular component? or perhaps you don't know the structure of your repository and you want to get to know it more intimately?

These screenshots show:

* The root of the repository, listing the currently defined distributions.
  Metadata associated with the distribution is also listed along with information such as the components available within the distribtuion and the total number of packages currently published within it.


![](https://github.com/andrewbunday/ReproWeb/raw/master/documentation/images/image_07.png)

* The breadcrumb which is left at the top of the interface as you delve deeper into the repository.

![](https://github.com/andrewbunday/ReproWeb/raw/master/documentation/images/image_08.png)

### Searching

> Do you have a specific package you are looking for? Do you know it's name or version and don't want to have to go browsing around for it?

The package list shows all of the packages currently available for a given architecture or all of the packages across all of the distributions in the repository. The search box can be used to search through these packages, as you type it will filter out those packages which don't match.

![](https://github.com/andrewbunday/ReproWeb/raw/master/documentation/images/image_05.png)

---

## Future Ideas

This is a list of the features which I'd like to add in time.

1. Make the package view accessable at an point in the browser, so for example
   allow listing of packages in a component.
1. Complete json endpoints for all urls.
1. Renaming or package or version number.
1. Listing of unreferrenced versions. Or previous versions which are no longer
   available in Releases.gz
1. Listing of pre/post install/remove scripts.

---

## Get hold of it yourself

I strongly recommending visiting [http://github.com/andrewbunday/reproweb](http://github.com/andrewbunday/reproweb) and start by reading the README and documentation on the repo.

Or for the impatient in life:

~~~ bash
# clone the repository

git clone https://github.com/andrewbunday/ReproWeb.git

# ensure that the cache directory has been created, or that you edit the setting in the configuration before you start up the app.

sudo mkdir /var/cache/reproweb

# start it up

python debug-run.py
~~~

---

## Technical Details

Underneath the hood ReproWeb is built using only a handful of _(at the time of writing)_ actively developed tools.

Development of Reproweb was conciously kept to a tight a deadline. We gave ourselves less than 2 weeks to go from establishing our initial requirements, having looked at the alternatives, to the first releaseable draft.

This drove our decisions behind each of the components we used to build the application.

* __reprepro__ - This is the repository management tool which the browser has been designed to be used with. All of the interactions with reprepro are handled through via the 3rd party script reepocheep.py which could be swapped out with an equivalent script which provides the same api.

Since we were using reprepro already to manage our software package repositories it made complete sense to tailor the app to its strengths.

The majority of the heavy lifting for the app is handled by calls to reprepro via subprocess and then handling of the returned data.

* __flask__ - A small web application backend framework which handles all of the functionality we need for it for this project. Something as feature rich as django would have been more cumbersome that the project required. Other parts of the Flask eco system were utilised, such as jinja2 to handle dynamic page generation.

This is one of my personally favourite frameworks to use when building these kinds of applications. Having worked previously with django and really having not gotten on well with it I've always been attracted to these lighter weight frameworks.

Another choice a few years ago might have been [web.py](http://webpy.org) which truely is a micro framework would have ended up being replaced as it reached its limits a little sooner than flask will.

* __bootstrap__ - A very user friendly web front-end framework which handles features such as responsive layout, navbar and general css styling consistency.

A few years ago if someone had told me that all of the css and jquery plugins I would need to build a webpage/app quickly with little stress I could download and get started with in minutes, I would have laughed and said 'your sooo funny'. However now Bootstrap does all those things and it has made the lives of non-web developers such as myself far easier.

* __datatables.js__ - A damn awesome jquery plugin which completely powers the tabulated, sortable, filterable, paginated list of packages seen in the search view.

Does what it says on the tin. Credit goes to [@PaulNendick](http://paulnendick.github.com) for the find.
