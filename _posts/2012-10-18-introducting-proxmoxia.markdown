---
date: 2012-10-18 10:27PM
layout: post
title: "Introducing Proxmoxia"
---

Today I'm happy to have finished up work on a small python wrapper for Proxmox's rest based API. We've called it [Proxmoxia](http://github.com/baseblack/proxmoxia).

###Background

For those who don't know, [Proxmox](http://pve.proxmox.com/wiki/Main_Page) is an open source server virtualization platform which allows you to create and manage any number of virtual machines based on either KVM or OpenVZ.

At Baseblack we utilise Proxmox to provide us with the majority of our central infrastructure. For example we run a pair of DNS servers, an almost countless number of license servers and farm manager all as openvz containers. Doing this provides us with a far higher level of service since if any one of our virtualization nodes go down we can quickly and pretty painlessly re-start the downed services on one of the other nodes whilst we work outs whats happening.

An area of active development for us is the automation of builds of new software packages for deployment onto our workstations, render nodes and virtual hosts.

> _This is where Proxmoxia comes in._ 

Until recently anytime we've needed to spin up a new virtual server, we've needed to login to the web interface and manually create a new instance. Not only is this tedious and time consuming, but it can also lead to the occasional mistake.

We knew that our plans for automated builds, and testing of packages would involve us spinning up more and more sacrificial virtual machines which we could use as testing/build bots, each being discarded once they had completed their assigned tasks.

###Creating Proxmoxia

After a short burst of _'Spanish Flea'_ our  preferred google search background music we found that the majority of the existing implementations of wrappers for the rest API had been in languages we either weren't keen on or flat out refused to use. 

> _Perl, Java, PHP need not apply_

Unfortunately I didn't come across PyProxmox _(a major omission of mine)_ until I'd already spent the better part of a day planning on how to make proxmoxia. And by then I well and truly had the bit between my teeth.

I wanted to create a library which followed a few simple rules:

1. It should 'feel' pythonic.
2. I wanted the wrapper to extend itself. Or in a few more words; I didn't want to have to personally write wrapper functions for each and every endpoint and method combination which are currently exposed in the rest api. 

Personally I don't like PyProxmox's implementation for both of these reasons which is why I feel justified in completing Proxmoxia. 

###Whats Next?

We're going to be using the library as part of our testing suite, creating new VMs off of the back of Jenkins builds. Hopefully it's going to work out to be really useful.

###Finally Some Examples

Okay, so here are some examples to show what I'm talking about. Head over to the [github](http://github.com/baseblack/proxmoxia) site for the source for these. 

First up, Proxmox uses a token based authentication mechanism which must be attached to the headers of each request. 

Connect to any node in the cluster and get the auth_token:

````python
import proxmox

connection = proxmox.Connector('proxmox-1', 8006)
auth_token = connection.get_auth_token('user@pam', 'strawberries')
````

Next create Proxmox and Node access objects, these are the bread and butter of the library:

````python
p = proxmox.Proxmox(connection)
node = proxmox.Node(connection, 'proxmox-7')  # this is on a different host.
````
    
The library translates its requests into the url equivalent of what you call. So here we request the status on a vm with id number 108:

````python
vmid = 108
print node.openvz(vmid).status.current()
````

Which is converted to the URL:

    http://proxmox-1:8006/api2/json/nodes/proxmox-7/openvz/108/status/current

Find a vm template filepath and use this as the ostemplate for a new vm you create. And then start it up once it has finished being created:

````python
# find the path for the template you want to use.
for template in node.storage('virtual-nfs').content(content='vztmpl'):
    if re.match('.*ubuntu-12.04-bb-20121010b_amd64.tar.gz$', template['volid']):
        volume = node.storage('virtual-nfs').content(template['volid']).get()

# create the container, giving it some sensible settings
taskid = node.openvz.post( ostemplate=volume.get('path'),
                           vmid=204,
                           hostname='test-4',
                           ip_address='192.168.123.204',
                           storage='local')

# keep an eye on task and see when its completed
while node.tasks(taskid).status()['status'] == 'running':
        time.sleep(1)

# print out the logs
for line in node.tasks(taskid).log():
    print line['t']

try:
    # start up the container
    node.openvz('204').status.start.post()
except:
    raise Exception('Unable to start container')
````

Find if a user exists and create them if they do not:

````python
if 'andrew.bunday@pve' not in [x['userid'] for x in p.access.users()]:
    p.access.users.post(userid='andrew.bunday@pve', comment="test user", password="strawberries")
````






