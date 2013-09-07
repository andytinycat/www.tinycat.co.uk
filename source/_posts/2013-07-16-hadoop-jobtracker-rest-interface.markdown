---
layout: post
title: "Hadoop JobTracker REST interface"
date: 2013-07-16 21:43
comments: true
categories: 
---

Hadoop's JobTracker and NameNode are very bad at presenting information about themselves in an easily consumable fashion.
Sure, you can probe them with [JMX](http://theholyjava.wordpress.com/2012/09/21/enabling-jmx-monitoring-for-hadoop-and-hive/), but
that limits your probing toolset to Java or something that runs on the JVM (e.g. JRuby, Jython, Scala, Clojure, etc). Plus,
it means dealing with the verbose and fairly opaque JMX interface.

For the JobTracker, it would be great if there was a simple REST API to query jobs and their properties.

To that end, I've written a very simple JRuby & Sinatra app that connects to the JobTracker using the Hadoop libraries,
and presents information about jobs in progress, queued, failed, etc via a simple REST interface, with JSON as the output format.

You can find it on [Github](https://github.com/forward3d/hadoop-jobtracker-rest). It was written for use with internal systems
at [Forward3D](http://forward3d.com).

I did plan to write something similar for the NameNode, but never got around to it. One day...
