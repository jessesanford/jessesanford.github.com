--- 
wordpress_id: 49
layout: post
title: On EIP, Camel, Spring and Object Recognition!?
date: 2010-04-15 08:34:29 -04:00
wordpress_url: http://www.jessesanford.com/?p=49
---
I have recently been doing some work with Camel routing library/engine for java that plays really nice with Spring and JMS via ActiveMQ. Camel makes it really easy to implement EIP (Enterprise Integration Patterns) into your Java app and ultimately allows you the beautiful flexibility describe in Gregor Hohpe's work on <a href="http://www.eaipatterns.com/">enterprise integration patterns</a>... great book btw!

Anyway I got to thinking how fun it would be to create a custom predicate that employs a machine learning algorithm (possibly based on a <a href="http://en.wikipedia.org/wiki/JOONE">neural network</a>) to do <a href="http://www.youtube.com/watch?v=N4m4j4D3pJU">image recognition</a> and route messages (images) based on it's understanding of them. Given enough time and enough input this could really open up some awesome possibilities!

The most immediately applicable solution is moderation, allowing the AI to do the first pass, pipe positives to an administrative interface where an actual human being could determine if the image was actually naughty or if the algorithm flagged a false positive. The feedback on false positives could then be driven back into the pipeline thus tuning the algorithm further (false positives) or deleting the image from the servers and warning the contributing user (actual positives).

Another cool application would be to do something like <a href="http://www.google.com/mobile/goggles">google is doing</a> where you could use the routing engine and the image recognition predicate to make a first guess at what the image is basically of and then query all sorts of different data stores/apis to find more information about the image. You wouldn't need to do so much legwork query everything or storing a "master index" you could simply allow the algorithm to make an educated guess and say for example "this is a painting" or "this is a barcode" and then ONLY query the applicable datasources.

Of course this will be full of false positives. BUT if you make it fun and allow feedback back into the routing engine from the end users the algorithm will get <a href="http://en.wikipedia.org/wiki/HAL_9000">smarter over time!</a>
