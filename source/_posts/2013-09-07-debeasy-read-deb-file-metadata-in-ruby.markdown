---
layout: post
title: "Debeasy - read .deb file metadata in Ruby"
date: 2013-09-07 20:12
comments: true
categories: 
---

As part of a script to manage apt-repos at $dayjob, I realised it would be nice to have a way to read
Debian package file metadata from the file without having to parse the output of `dpkg` and friends.

Enter [Debeasy](https://github.com/andytinycat/debeasy).

It's a very simple gem to extract package metadata. You can also find it on [Rubygems](https://rubygems.org/gems/debeasy).
