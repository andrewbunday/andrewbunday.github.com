---
date: 2012-12-04 8:45AM
layout: post
title: "Running Puppet without a Master Server"
tags: puppet
navigation: true
cover: assets/images/posts/2012-12-04-cover.jpg
class: post-template
subclass: 'post tag-puppet'
author: andrew
---

Typically the easiest way to run puppet is to use the default master and agent
mechanism.

When you run puppet using a master the only way to switch between
different sets of manifests/modules is to use environments. If  you setup puppet
correctly these can allow you to pick and choose which environment you want to
use on a given run, and/or for a given node.

Unfortunately there are a few caveats when using environments.

* To pick out a particular enviroment for a specific node each time you ideally
  should be using an ENC. Once you setup your enviroment == node relationship it'll
  stick for each run.
* If you are not using an ENC, since this is another server which you don't want
  to have to maintain then you are requried to specify the environment to use
  via the commandline. For example:

  > `puppet agent --test --environment canary_maya_001`
* Each environment will need to have a different manifest/modulepath defined and
  located in your puppet manifests.

### Running Masterless

Another way of running puppet is to setup it up in in a masterless mode. Here we
instead checkout a copy of our manifest from a central repository and run puppet
directly against it.

Whilst you gain the advantage of not needing a master, and of being able to split
environments more easily with branches (this can be done with the master but requires
multiple checkouts of the puppet repository). You do loose out on a few features
of puppet, such as:

* Reports, these really don't work as well.
* StoreConfigs is probably the biggest feature loss although you should be able
  to solve the problems you would have used it for in other ways (SSHFP for example
  when handling new server certificates).
* To use a particular environment on a node it becomes easier to create branches
  of your manifests/modules and then checkout the branch you want on the node
  just before you execute puppet. This still means you have multiple 'copies' of
  files, but managing them tends to be easier via standard git branch&merge tools.

### An Example Wrapper Script

The steps involved in running masterless are slightly more cumbersome than
running with a master. In particular if you use a git server such as gitolite
then you need to ensure that you have credentials setup which allow you to read
from the repository.

For this we're using a locally served ssh config file and private rsa id which
the config file informs ssh to use to identifty ourselves when we contact our
gitolite host.

_served with a pinch of salt_

{% highlight bash linenos %}
#!/usr/bin/env bash

# check if sudo has been used or user is root
if [ -z "${SUDO_USER}" ] && [ "${USER}" != "root" ]; then
    echo "Fail: Only root can execute puppet"
    exit 1
fi

# change to 'syslog' to redirect puppet's internal logging directly to syslog
LOGDEST='console'

# this may be overkill. A time/datestamp or even standard path may be more
# suitable.
UUID=`uuidgen`

# SSH credentials setup
if [[ ! -d $HOME/.ssh ]]; then
    mkdir --mode=700 $HOME/.ssh
fi

# Stash $HOME/.ssh/config if it already exists. We'll put it back when we're
# finished.
if [ -f $HOME/.ssh/config ]; then
    mv $HOME/.ssh/config $HOME/.ssh/$UUID.config
fi

test -x /usr/bin/wget || exit 1
wget -q -O $HOME/.ssh/config http://autoserver/d-i/bin/puppet_ssh.config
wget -q -O $HOME/.ssh/puppet_readonly_id_rsa \
           http://autoserver/d-i/bin/puppet_readonly_id_rsa

# OpenSSH will bork out if the permissions are not exactly right.
chmod 0600 $HOME/.ssh/puppet_readonly_id_rsa
chmod 0600 $HOME/.ssh/config

# Clone a copy of the puppet manifests
if [ ! -d /var/tmp/puppet/$UUID/.git ]; then
    git clone gitlab:puppet.git /var/tmp/puppet/$UUID
fi

# Get any updates to a previously checked out manifest repo
if [ -f /var/tmp/puppet/$UUID/manifests/site.pp ]; then
    git --git-dir=/var/tmp/puppet/$UUID/.git \
        --work-tree=/var/tmp/puppet/$UUID fetch
    git --git-dir=/var/tmp/puppet/$UUID/.git \
        --work-tree=/var/tmp/puppet/$UUID merge origin/master
fi

# Return the users .ssh settings.
if [ -f $HOME/.ssh/$UUID.config ]; then
    rm $HOME/.ssh/config
    mv $HOME/.ssh/$UUID.config $HOME/.ssh/config
    chmod 0600 $HOME/.ssh/config
fi

# Finally run puppet. This *can* take a while.
puppet apply --modulepath=/var/tmp/puppet/$UUID/modules --color false\
       --vardir /var/tmp/puppet/var --confdir /var/tmp/puppet/etc --no-report \
       --logdest $LOGDEST /var/tmp/puppet/$UUID/manifests/site.pp ${1}
{% endhighlight %}
