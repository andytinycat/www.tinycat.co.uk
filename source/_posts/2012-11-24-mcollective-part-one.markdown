---
layout: post
title: "MCollective - Part One"
date: 2012-11-21 13:21
comments: true
categories: sysadmin
---

In this post I'm going to talk about MCollective, publish-subscribe middleware, the architecture, and the terminology involved; this confused me when I started exploring what MCollective could do for me.

## Configuration Management vs. Server Orchestration

So everyone agrees that modern Configuration Management (CM) is a good thing. Be it Puppet, Chef, or even CFEngine, the modern DevOp wouldn't be seen dead without some form of CM doing the heavy lifting of configuring machines. CM is great for two things:

* Taking a minty-fresh, just-created VM and making it like some other machine.
* Making sure that machine and all its brethren stay in the same state.

However, more and more you will find yourself needing to perform some time-sensitive task on a number of systems at once, such as:

* Push out a change to configuration on a bunch of boxes in a farm, Right Now, and be notified of those that don't respond or don't get the update.
* Work out what machines are vulnerable to some new and dangerous remote-root exploit.
* Push a small job to some machines to fix a problem, restart a service, or simply to check everything's okay. (Spoiler alert: it won't be. Life's like that.)

That's where your server orchestration tool comes in. It can work alongside Puppet or Chef; in fact, one of the first reasons you'll find yourself reaching for just such a tool is that you think "Hey, it'd be awesome if I could get Puppet or Chef to run right this second." You may even want to ditch running a daemonized Puppet due to its somewhat mercurial nature while daemonized. That's just the start, though.

## What's wrong with what we have?

There's a lot of tools in the system automation space (Func, Capistrano, Fabric, Salt, even parallel SSH if you're desperate/stuck in the 90s). Personally, I trialled Func, Fabric and Salt (Capistrano is too heavily focussed on deploying webapps to be truly general purpose). I found these three tools failed in three areas:

* They are effectively parallel SSH + metadata to help you discover hosts. You still end up using them like parallel SSH, by invoking their command line module (and they all have one).
* They all make it quite hard to deal with the situation where one of your hosts isn't responding for some reason (daemon's stopped, machine's hung, or it's just not your day). That's a killer when you're trying to push something that *must* reach all live systems. I want my orchestration tool to make it configurably *obvious* that my task didn't hit every machine I wanted it to, and make it *easy* for me to check who got it and who didn't.
