--- 
wordpress_id: 73
layout: post
title: How to increase your .bash_history size a.k.a. what to do when you suffer from short term command line syntax loss
date: 2010-10-12 14:25:05 -04:00
wordpress_url: http://www.jessesanford.com/?p=73
---
vi ~/.bashrc 
add 
HISTSIZE=1000
(default is 500 on most *nix)
then
source ~/.bashrc
echo $HISTSIZE
To see the old size
echo $HISTFILESIZE

If you really want to get tricky you can set your .bash_history size REALLY high and then use logrotate to make sure you don't fill up your hd.
See here:
http://linux.derkeiler.com/Mailing-Lists/SuSE/2008-12/msg00864.html
