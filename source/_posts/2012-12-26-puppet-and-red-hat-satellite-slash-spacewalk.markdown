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
