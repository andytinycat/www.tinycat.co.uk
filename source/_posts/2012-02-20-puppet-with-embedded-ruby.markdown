---
layout: post
title: "Puppet with embedded Ruby"
date: 2012-02-20 22:52
comments: true
categories: sysadmin
---

One thing that sucks about deploying Ruby apps is that your configuration management tool (e.g. Puppet) is usually also written in Ruby, which means it can't manage the installed Ruby version easily. Also, you may get into a situation where your app requires one version of Ruby, and Puppet requires another.

Opscode's approach to this with Chef is to ship a completely standalone Chef, with all its dependencies completely built into a single package, running from an embedded Ruby interpreter. They called this (Omnibus packaging)[http://www.opscode.com/blog/2012/06/29/omnibus-chef-packaging/].

(Omnibus)[https://github.com/opscode/omnibus-ruby] is quite easy to pick up and use, so I've used this to create a Puppet Omnibus package. It comes with all the gems required for Puppet to run, and Ruby 1.9.3 to run it. You can find the Github repo for building your own package (in my repository on Github)[https://github.com/andytinycat/puppet-omnibus].

I've taken the approach of not building all binary dependencies from source; instead, I have the final OS package require other OS packages containing the binary dependencies it requires (which amounts to OpenSSL, libaugeas and a few others). This saves a bit of time and complexity, but at the potential of a breaking change in those libraries screwing up the Omnibus package. It's entering production now, so we'll see how it goes.

At some point in the future I may host a package repo for it.
