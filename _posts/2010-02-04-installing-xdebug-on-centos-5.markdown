--- 
wordpress_id: 45
layout: post
title: Installing xdebug on centos 5
date: 2010-02-04 20:36:59 -05:00
wordpress_url: http://www.jessesanford.com/?p=45
---
Performance profiling php in my opinion is best done with xdebug and kcachegrind (install it via APT on an Ubuntu virtual machine and save yourself some trouble. I know you can get it to work with xwindows natively on a mac but damn it's not easy and it certainly isn't as pretty as using it in KDE 4 )

How toÂ performanceÂ profile an app is an art not covered here. Seriously if you want to start somewhere I suggest reading this article over at Zend developer zone (It's from 2007 but it's still relevant):

<a href="http://devzone.zend.com/article/2899-Profiling-PHP-Applications-With-xdebug">http://devzone.zend.com/article/2899-Profiling-PHP-Applications-With-xdebug</a>

Also there is a great article if you can find it from the September 2004 edition of php|Architect... If you can't find it on the internet email me and I will send you a PDF version of it. It is a complete and comprehensive article and should not be missed if you are working with xdebug at all.

So finally back on topic:

Installing xdebug is pretty easy assuming you have php/pecl/pear installed... If you don't I suggest you use yum and if you don't have that installed then just follow this tutorial: <a href="http://www.matteomattei.com/en/install-yum-and-php-pear-on-centos-5">http://www.matteomattei.com/en/install-yum-and-php-pear-on-centos-5</a>

Anyway once you have pecl installed you can run the following:
<blockquote># pear install pecl/xdebug

downloading xdebug-2.0.5.tgz ...
Starting to download xdebug-2.0.5.tgz (289,234 bytes)
............................................................done: 289,234 bytes
67 source files, building
running: phpize
Configuring for:
PHP Api Version: Â  Â  Â  Â  20041225
Zend Module Api No: Â  Â  Â 20050922
Zend Extension Api No: Â  220051025
/usr/bin/phpize: /tmp/tmpOcEvNL/xdebug-2.0.5/build/shtool: /bin/sh: bad interpreter: Permission denied
Cannot find autoconf. Please check your autoconf installation and the $PHP_AUTOCONF
environment variable is set correctly and then rerun this script.</blockquote>
Woops!

Looks like something is up with running /tmp/tmpOcEvNL/xdebug-2.0.5/build/shtool

I have seen this type of build error before when using other package management systems and it turns out that what I thought was true. That the /tmp directory was mounted with the noexec switch.

See here:
<blockquote># mount -l | grep /tmp
simfs on /tmp type simfs (rw,noexec)
simfs on /var/tmp type simfs (rw,noexec)</blockquote>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;"># pear install pecl/xdebug</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">downloading xdebug-2.0.5.tgz ...</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">Starting to download xdebug-2.0.5.tgz (289,234 bytes)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">............................................................done: 289,234 bytes</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">67 source files, building</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">running: phpize</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">Configuring for:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">PHP Api Version: Â  Â  Â  Â  20041225</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">Zend Module Api No: Â  Â  Â 20050922</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">Zend Extension Api No: Â  220051025</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">/usr/bin/phpize: /tmp/tmpOcEvNL/xdebug-2.0.5/build/shtool: /bin/sh: bad interpreter: Permission denied</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">Cannot find autoconf. Please check your autoconf installation and the $PHP_AUTOCONF</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">environment variable is set correctly and then rerun this script.</div>
So let's remount it with the exec switch:
<blockquote>mount -o remount,exec /tmp</blockquote>
And now let's see what mount -l says:
<blockquote>mount -l | grep /tmp
simfs on /tmp type simfs (rw)
simfs on /var/tmp type simfs (rw,noexec)</blockquote>
So we can see that now the /tmp does not have noexec listed.

Ok let's try and install via pecl again:
<blockquote># pear install pecl/xdebug
downloading xdebug-2.0.5.tgz ...
Starting to download xdebug-2.0.5.tgz (289,234 bytes)
............................................................done: 289,234 bytes
67 source files, building
running: phpize
Configuring for:
PHP Api Version: Â  Â  Â  Â  20041225
Zend Module Api No: Â  Â  Â 20050922
Zend Extension Api No: Â  220051025
building in /var/tmp/pear-build-root/xdebug-2.0.5
running: /tmp/tmp3tHVR2/xdebug-2.0.5/configure
checking for egrep... grep -E
checking for a sed that does not truncate output... /bin/sed
checking for gcc... gcc
checking for C compiler default output file name... a.out
checking whether the C compiler works... configure: error: cannot run C compiled programs.
If you meant to cross compile, use `--host'.
See `config.log' for more details.
ERROR: `/tmp/tmp3tHVR2/xdebug-2.0.5/configure' failed</blockquote>
Well were not in the clear yet the error looks surprisingly similar... let's just try to mount /var/tmp without the noexec switch as well:
<blockquote># mount -o remount,exec /var/tmp</blockquote>
And again the pecl install:
<blockquote># pear install pecl/xdebug
downloading xdebug-2.0.5.tgz ...
Starting to download xdebug-2.0.5.tgz (289,234 bytes)
..................................................done: 289,234 bytes
67 source files, building
running: phpize
Configuring for:
PHP Api Version: Â  Â  Â  Â  20041225
Zend Module Api No: Â  Â  Â 20050922
Zend Extension Api No: Â  220051025
building in /var/tmp/pear-build-root/xdebug-2.0.5
running: /tmp/tmpkWolpH/xdebug-2.0.5/configure
checking for egrep... grep -E
checking for a sed that does not truncate output... /bin/sed
checking for gcc... gcc
checking for C compiler default output file name... a.out
checking whether the C compiler works... yes
checking whether we are cross compiling... no
checking for suffix of executables...

&lt;SNIP&gt;

Build process completed successfully
Installing '/var/tmp/pear-build-root/install-xdebug-2.0.5//usr/lib64/php/modules/xdebug.so'
install ok: channel://pecl.php.net/xdebug-2.0.5
You should add "extension=xdebug.so" to php.ini</blockquote>
<div>Nice. Ok so now it worked. Notice the path that it was installed to (on this 64 bit machine it was):Â /usr/lib64/php/modules/xdebug.so</div>
<div>We will need that later when we put it in our new /etc/php.d/xdebug.ini: (the contents of which I will explain in another blog post but for now the comments inside of it should be self explanatory)</div>
<blockquote>#extension_dir = "/usr/local/lib/php/20060613"

#doc_root="/usr/local/apache2/htdocs/web"

#the following was added by jesse to fix the path variables when run in cgi mode
#cgi.fix_pathinfo=0

zend_extension="/usr/local/lib/php/20060613/xdebug.so"
#xdebug.remote_enable=1

;JESSE: the following was taken from: http://code.google.com/p/syslogr-utils/wiki/XdebugHelper
;When using Eclipse with PDT and xdebug -- make sure to change your
;tools --&gt; addons --&gt; xdebug helper --&gt; preferences --&gt; idekey = ECLIPSE_DBGP
;(This is the setting that XDEBUG_SESSION_START is set to on your web browser URL)

;JESSE: I have put the following notes in from some research into profiling with webgrind:
;(they were taken from: http://www.chrisabernethy.com/php-profiling-xdebug-webgrind/ )

;NOTE: usually use cachegrind.out.%t.%p when I want one output file per script run,
;but if I want to run a script multiple times and see the aggregate numbers
;I use cachegrind.out.%s and set xdebug.profiler_append = on

;NOTE: A Nice to know, for different cachegrind files per host you can use %H in the
;profiler_output_name, according to
;http://www.xdebug.org/docs/all_settings#trace_output_name

;Always profile scripts with xdebug:
;xdebug.profiler_enable = 1

#xdebug.profiler_enable=1

;Alternatively, enable profiling with GET/POST parameter XDEBUG_PROFILE,
;e.g. http://localhost/samplepage.php?XDEBUG_PROFILE:
;xdebug.profiler_enable_trigger = 1

xdebug.profiler_enable_trigger=1

xdebug.profiler_output_dir="/tmp/xdebug/"

;the following forces xdebug to append to the outfile rather than
;overwrite it on each exec of a script
;xdebug.profiler_append=On

;the patterns for the names of the outfiles
;xdebug.profiler_output_name = cachegrind.out.%s
xdebug.profiler_output_name = cachegrind.out.%t.%p

;the following opens debug output links in textmate
xdebug.file_link_format = "txmt://open?url=file://%f&amp;line=%l"

xdebug.remote_host="127.0.0.1"
xdebug.remote_port=9000
xdebug.remote_handler="dbgp"
xdebug.remote_mode=req
xdebug.idekey=1</blockquote>
Now check the php-cli and see if xdebug shows up in it's ini dump.
<blockquote>php -i | grep xdebug
/etc/php.d/xdebug.ini,
xdebug
xdebug support =&gt; enabled
xdebug.auto_trace =&gt; Off =&gt; Off
xdebug.collect_includes =&gt; On =&gt; On
xdebug.collect_params =&gt; 0 =&gt; 0
xdebug.collect_return =&gt; Off =&gt; Off
xdebug.collect_vars =&gt; Off =&gt; Off
xdebug.default_enable =&gt; On =&gt; On
xdebug.dump.COOKIE =&gt; no value =&gt; no value
<div>&lt;snip&gt;</div></blockquote>
Looking good!

Oh and obviously restart apache... /etc/init.d/httpd restart

Sweet.
