--- 
wordpress_id: 90
layout: post
title: Bash Background Exec aka daemonize your executable
date: 2011-07-26 16:08:38 -04:00
wordpress_url: http://www.jessesanford.com/?p=90
---
I was having trouble getting a node process to work correctly with start-stop-daemon's --background switch and also allowing for me to pipe stderr and stdout wherever I deemed necessary.

Turns out that the --backround switch expects the process being daemonized to take care of it's own logging and does not allow access to the stderr and stdout file descriptors as you would normally expect.

SO I bring you Bash Background Exec

&lt;SNIP&gt; Waiting for license review SORRY! I will post a very detailed english explination of the solution shortly!&lt;/snip&gt;

With it you can daemonize whatever executable you want and pass it parameters like you normally would while also piping stderr and stdout someplace nice. Here is an example of backgrounding a node process.
<pre><code>$ ./bbexec.sh -d /usr/local/bin/node -p /var/run/node -l \ 
</code><span class="Apple-style-span" style="font-family: monospace;">&gt; /var/log/node -a "app.js --debug-brk"</span></pre>
Remember the above is most useful when being called from an init.d script utilizing start-stop-daemon. I will be adding an example init.d script for node shortly.

Check back soon!

P.S. I am fully aware that this does not actually Daemonize in the strict sense. In particular it violates (on purpose) the following rule of Daemons:
<blockquote>"Closing all inherited open files at the time of execution that are left open by the parent process, including file descriptors 0, 1 and 2 (stdin, stdout, stderr). Required files will be opened later." -<a title="wikipedia: Daemon (computing)" href="http://en.wikipedia.org/wiki/Daemon_(computing)" target="_blank">http://en.wikipedia.org/wiki/Daemon_(computing)</a></blockquote>
