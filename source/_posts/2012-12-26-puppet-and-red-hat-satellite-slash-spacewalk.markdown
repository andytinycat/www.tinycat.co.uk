---
layout: post
title: "Puppet and Red Hat Satellite/Spacewalk"
date: 2012-12-26 22:13
comments: true
categories: sysadmin
---

The place I currently work at recently decided that we would standardise the Linux distributions we had in use, with the aim being to use only Red Hat Enterprise Linux wherever possible. To deliver packages, we ended up with RHN Satellite installed on a VM. Now, we provision pretty much everything possible with Puppet, and Satellite required clicky-clicky in the web interface to configure machines. That's not how I like to do things.

## A note on Satellite/Spacewalk

In case you're confused, Spacewalk is the open-sourced code that makes up RHN Satellite, with the branding removed. The module described below will work with Spacewalk as well as Satellite. (In theory: I haven't tested it.

## Assigning machines to Satellite channels with Puppet

Satellite is weird, in that you assign machines to "channels" (repositories) on the Satellite server itself; RHEL clients running yum ask the Satellite: "Hey, what channels am I subscribed to?", and Satellite returns whichever ones the client is permitted to use. With bog-standard yum repositories, you configure the client to look at a URL, and that's it.

The first challenge was figuring out how a RHEL *client* can change the list of channels it is currently subscribed to. The Satellite API doesn't make any mention of a mechanism for doing this, but there's a Python script called `spacewalk-channel` which is part of the `rhn-setup` package, which can change a client's channel subscriptions *from the client*.

This code makes use of an API that doesn't appear in the Satellite API docs, called `up2date`. It was pretty simple to figure out how this API works, and engineer a Puppet type & provider that hooks it.

The code is on [Github](https://github.com/andytinycat/puppet-rhnsatellite). It can't handle changing base channel; only child channel subscriptions. I do have a mechanism for doing this in our environment, but it needs cleaning up before it can be made public (it's a bit too specific for the way we do things). It also doesn't register the machine against Satellite, though that should be trivial for you to script - in my environment, our provisioning steps do the registration and install Puppet, at which point the module takes over.

## Ideas for expansion

One thing I'd really like this module to do is read the child channel's configuration, and see if it has a GPG key associated with it, then bring the key down to the system and install it. At the moment, I have a separate module that installs all the extra GPG keys for packages we require, and it's ugly - to do this, I use stschulte's [rpmkey type](https://github.com/stschulte/puppet-rpmkey).
