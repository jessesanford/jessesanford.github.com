--- 
wordpress_id: 59
layout: post
title: "Undertstanding Apache HTTPD mod_fcgid variables: PHP via suexec fastcgi"
date: 2010-07-27 09:46:32 -04:00
wordpress_url: http://www.jessesanford.com/?p=59
---
<p>
Had to talk through another php mod_fcgid setup today. It is amazing to me how difficult it is to find clear information on this setup out there on the internet.
</p>
<p>
If any of the below are wrong please tell me. I am not always right but I have studied this setup quite a bit and have come across many permutations and the below finally make sense.
</p>
<h4>
Assumptions:
</h4>
<p>
The most MEMORY efficient way to use php with apache is via the WORKER MPM with a fastcgi plugin forking PHP-CGI processes. I specify MEMORY here because I am pretty sure that CPU wise mod_php is still faster. I can only assume that is because all of the data structures etc that are used within php are within direct access of the apache processes servicing the request. Faster still is probably compiling mod_php directly into apache rather than using the shared module.
</p>
<p>
The most MEMORY efficient way of using PHP-CGI is to use as FEW parent processes as possible if not 0 and use as many child processes as possible under those parent processes. This is especially true when using an opcode cache like APC. There is a known bug with the mod_FCGID that causes it to be unable to share memory between FCGI processes. mod_FASTCGI on the otherhand does not have this bug. Why not use mod_FASTCGI module instead? Because unfortunately it is a dead project (nothing since 2003): http://freshmeat.net/projects/mod_fastcgi/
</p>
<p>
Thus APC is not very effective when using mod_fcgid. It just can't share its memory across subsequent requests to the same php code if those requests get routed to seperate php-cgi processes. See here:
</p>
<p>
"PHP child process management (PHP_FCGI_CHILDREN) should always be disabled with mod_fcgid, which will only route one request at a time to application processes it has spawned; thus, any child processes created by PHP will not be used effectively. (Additionally, the PHP child processes may not be terminated properly.) By default, and with the environment variable setting PHP_FCGI_CHILDREN=0, PHP child process management is disabled.
</p>
<p>
The popular APC opcode cache for PHP cannot share a cache between PHP FastCGI processes unless PHP manages the child processes. Thus, the effectiveness of the cache is limited with mod_fcgid; concurrent PHP requests will use different opcode caches."
</p>
<p>
Anyway I used the following config on a relatively low traffic 4gb virtual instance with a VERY memory intensive PHP app.
</p>
<h4>
In apache httpd.conf:
</h4>
<code>
<p>
&lt;IfModule mod_fcgid.c>
AddHandler    fcgid-script .fcgi
&lt;/IfModule>
</p>
<p>
## Sane place to put sockets and shared memory file
#FcgidIPCDir run/mod_fcgid #to use sockets on unix or pipes on windows
#FcgidProcessTableFile run/mod_fcgid/fcgid_shm #to use shared memory on unix
</p>
</p>
&lt;IfModule mod_fcgid.c>
</p>
<p>
## FcgidInitialEnv PHPRC "/etc" # is also set in the wrapper script per virtual host. both ways will send the variables on to the cgi instances
##the following MaxRequestsPerProcess is also set in the wrapper script. both ways will send the variables on to the cgi instances
</p>
<p>
FcgidInitialEnv PHPRC=/etc/php5/cgi #working directory of php-cgi
</p>
<p>
#"By default, PHP FastCGI processes exit after handling 500 requests, and they may exit after this module has already connected to the application and sent the next request. When that occurs, an error will be logged and 500 Internal Server Error will be returned to the client. This PHP behavior can be disabled by setting PHP_FCGI_MAX_REQUESTS to 0, but that can be a problem if the PHP application leaks resources. Alternatively, PHP_FCGI_MAX_REQUESTS can be set to a much higher value than the default to reduce the frequency of this problem. FcgidMaxRequestsPerProcess can be set to a value less than or equal to PHP_FCGI_MAX_REQUESTS to resolve the problem."
</p>
<p>
FcgidInitialEnv PHP_FCGI_MAX_REQUESTS 5000 #same as below just as environment variable. sometimes good to set as well. due dilligence
</p>
<p>
FcgidMaxRequestsPerProcess 5000 #restarts child after so many requests to take care of memory leaks. needs to equal or be less than the PHP_FCGI_MAX_REQUEST variable sent to the cgi process
</p>
<p>
FcgidIdleTimeout 300 #allows process to sit for 5 mins while idle
FcgidIdleScanInterval 120 #how often fcgid parent polls children who are idle
FcgidBusyTimeout 3000 #allows php to spin for 50 mins before throwing 500 error
FcgidBusyScanInterval 120 #how often fcgid parent polls children who are busy
FcgidErrorScanInterval 60 #how often fcgid parent polls children for errors
FcgidZombieScanInterval 60 #how often to scan for zombie processes
FcgidProcessLifeTime 7200 #max time a child is alive by default
FcgidMaxProcesses 15 #max children PER PARENT PROCESS (remember this is not max TOTAL)
FcgidMaxProcessesPerClass 15 #Max fcgi processes
FcgidMaxProcessesPerClass 15 #Min to leave lying around
FcgidIPCConnectTimeout 3600 #Max time to wait for first "packet" on port or socket from child process
FcgidIPCCommTimeout 3600 #Max time to wait since the last  "packet" on port or socket from child process
FcgidOutputBufferSize 128 #comm buffer between mod_fcgid and the cgi process. flushed to client after full.
</p>
<p>
&lt;/IfModule>
</p>

<h4>
In vhost definition:
</h4>
<p>
&lt;VirtualHost *:80>
&lt;IfModule mod_fcgid.c>
</p>
<p>
SuexecUserGroup some_unprivileged_user_with_rights_to_webroot some_unprivileged_group_with_rights_to_webroot   #often these are www or apache user and group
</p>
<p>
FcgidFixPathinfo 1 #"This directive enables special SCRIPT_NAME processing which allows PHP to provide additional path information. The setting of FcgidFixPathinfoshould mirror the cgi.fix_pathinfo setting in php.ini."
</p>
<p>
&lt;Directory "/var/www/html">
Options +ExecCGI
AllowOverride All
AddHandler fcgid-script .php
</p>
<p>
FcgidWrapper /var/www/php-fcgi-scripts/php-fcgi-starter .php #location of your suexeced bash script
</p>
<p>
Order allow,deny
Allow from all
&lt;/Directory>
</p>
<p>
&lt;/IfModule>
&lt;/VirtualHost>
</p>

<h4>
Inside the suexec starter script /var/www/php-fcgi-scripts/php-fcgi-starter
</h4>
<p>
#!/bin/sh
PHPRC=/etc/php5/cgi/ #again the php-cgi working directory
export PHPRC #also set in the httpd.conf. I think this just stomps the one set in the httpd.conf.
</p>
<p>
#"By default, PHP FastCGI processes exit after handling 500 requests, and they may exit after this module has already connected to the application and sent the next request. When that occurs, an error will be logged and 500 Internal Server Error will be returned to the client. This PHP behavior can be disabled by setting PHP_FCGI_MAX_REQUESTS to 0, but that can be a problem if the PHP application leaks resources. Alternatively, PHP_FCGI_MAX_REQUESTS can be set to a much higher value than the default to reduce the frequency of this problem. FcgidMaxRequestsPerProcess can be set to a value less than or equal to PHP_FCGI_MAX_REQUESTS to resolve the problem."
</p>
<p>
export PHP_FCGI_MAX_REQUESTS=5000 #again setting the max number of requests per child process. Again also in httpd.conf so I think these are redundant.
</p>
<p>
#"PHP child process management (PHP_FCGI_CHILDREN) should always be disabled with mod_fcgid, which will only route one request at a time to application processes it has spawned; thus, any child processes created by PHP will not be used effectively. (Additionally, the PHP child processes may not be terminated properly.) By default, and with the environment variable setting PHP_FCGI_CHILDREN=0, PHP child process management is disabled. The popular APC opcode cache for PHP cannot share a cache between PHP FastCGI processes unless PHP manages the child processes. Thus, the effectiveness of the cache is limited with mod_fcgid; concurrent PHP requests will use different opcode caches."
</p>
<p>
export PHP_FCGI_CHILDREN=0 #THIS IS WHERE YOU SET THE NUMBER OF PARENT PROCESSES These will each spawn 15 children in this example. If set to 0 then you will have 1 parent with 15 children and apache will handle spawning the children not php.
</p>
<p>
exec /usr/bin/php-cgi #the php-cgi binary
</p>
</code>
<h4>
References:
</h4>
<a href="http://www.linode.com/forums/viewtopic.php?t=2982&postdays=0&postorder=asc&start=15">http://www.linode.com/forums/viewtopic.php?t=2982&postdays=0&postorder=asc&start=15</a><br/>
<a href="http://www.magentocommerce.com/boards/viewthread/29264/">http://www.magentocommerce.com/boards/viewthread/29264/</a><br/>

<a href="http://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html">http://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html</a><br/>

<a href="http://2bits.com/articles/apache-fcgid-acceptable-performance-and-better-resource-utilization.html">http://2bits.com/articles/apache-fcgid-acceptable-performance-and-better-resource-utilization.html</a><br/>
