---
layout: default
author: Cameron McKenzie
title: What is Java Mission Control?
blurb: A quick definition of what Java Mission Control is
---
<div class="embed-responsive embed-responsive-16by9">
<iframe width="560" height="315" src="https://www.youtube.com/embed/E3gxhuATmHs" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

# What is Java Mission Control?

Java Mission Control is a Java performance monitoring tool by Oracle which has been shipped with the JDK since Java version 7 update 40. It encompasses two JMX Console and Java Flight Recorder. Java Mission Control works by interacting with a JMX agent in a JVM which has an MBean server that integrates with the built in VM and app instrumentation running the in the JVM. That’s a key piece as it does lower the usage cost of the tool as it’s using pre-existing hooks. Oracle state this is normally well below a 1% performance cost.

Flight recorder, with its historical view, provides the ability to see trends in your JVM. This gives you the data needed to find memory leaks, latency issues around thread waits, locking issues and more. The unique feature Java Mission Control brings to the table is that it’s shipped with the Oracle JDK. There’s nothing that you need to install or attach to your existing VM to get it working. Start up a terminal go to your JDK bin directory and just type jmc.


