---
layout: post
title: "Ruby DAAP server"
date: 2013-02-19 20:07
comments: true
categories: 
---

DAAP is Apple's closed music-sharing/streaming protocol that's embedded in iTunes and other media products.

My general experience with iTunes music sharing is that it works pretty well. At work I use OpenVPN to tunnel back into my house network, since iTunes has the restriction of only allowing libraries to be shared on a local network.

I've previously used [forked-daapd](https://github.com/jasonmc/forked-daapd "Forked Daapd") and [Tangerine](https://launchpad.net/tangerine "Tangerine") as my DAAP server. However, I've never been fully happy with either, and I thought it would be an interesting Ruby project to write a DAAP server from scratch.

The fruits of my labour is the (un)imaginatively titled [rubydaap](https://github.com/andytinycat/rubydaap "Rubydaap").

It's a simple Sinatra app which uses Thin as the Rack webserver. The database is MongoDB, because it's fast, simple to use, and the DAAP protocol doesn't require us to consider data in a relational way. I made use of [dmap-ng](https://github.com/chendo/dmap-ng "dmap-ng") to provide the DAAP protocol parsing, which saved a bunch of time. It also makes use of the excellent [Taglib bindings](https://github.com/robinst/taglib-ruby "taglib-ruby") for Ruby. It makes use of inotify on Linux (and equivalents on other platforms) to look for file changes after the initial scan.

It's a bit broken right now, as it needs to be a bit more careful about how it reads files in the Scanner thread - it's possible to read MP3 files with funky tags that'll cause the app to stop. If you have a reasonably sensible music collection, it should work pretty well. When I get a free day or so, I'll make the Scanner much more robust (I've managed to crash Ruby reading a bad MP3 file's tags using taglib-ruby!)
