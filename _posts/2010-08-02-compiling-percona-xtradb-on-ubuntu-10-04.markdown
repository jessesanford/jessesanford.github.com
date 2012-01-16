--- 
wordpress_id: 67
layout: post
title: Compiling percona xtradb on ubuntu 10.04
date: 2010-08-02 08:35:15 -04:00
wordpress_url: http://www.jessesanford.com/?p=67
---
I recently had to compile from source the Percona blend of mysql 5.4 and ran into a few hiccups that I don't normally see when doing a vanilla mysql install. In particular I ran into the same issue described <a href="http://phaq.phunsites.net/category/faq/operating-systems/freebsd/">here</a> and <a href="http://ronaldbradford.com/blog/problems-compiling-mysql-54-2009-06-11/">here</a>.
<code>
"/bin/bash ../ylwrap sql_yacc.yy y.tab.c sql_yacc.cc y.tab.h sql_yacc.h y.output sql_yacc.output --   -d --verbose
../ylwrap: line 111: -d: command not found
make[1]: *** [sql_yacc.cc] Error 1"
</code>
 Apparently having nothing to do with percona but with mysql and yacc not finding bison and possibly flex. So I installed both of those via APT and ran a make clean, make, make install and still same error. Then I realized that the configure script is probably what sets the make switches for using bison so I re-ran 
<code>
CC="gcc -static-libgcc" CFLAGS="-O3 -pipe -m64 -fPIC -fomit-frame-pointer" CXX="gcc -static-libgcc" CXXFLAGS="-O3 -pipe -m64 -fPIC -fomit-frame-pointer -felide-constructors -fno-exceptions -fno-rtti" sudo ./configure -prefix=/opt/xtradb -enable-local-infile -enable-thread-safe-client -enable-assembler -with-client-ldflags=-all-static -with-mysqld-ldflags=-all-static -with-unix-socket-path=/opt/xtradb/tmp/mysql.sock -with-plugins=partition,archive,blackhole,csv,federated,heap,ibmdb2i,innobase,innodb_plugin,myisam,myisammrg -with-big-tables -without-debug -with-tcp-port=3306 -with-readline -enable-profiling -disable-shared -enable-static -with-extra-charsets=complex -with-pic -with-fast-mutexes -with-zlib-dir=bundled -with-ssl
</code>
and then another
<code>make</code>
<code>make install</code>

And it worked successfully.

Hope this helps!

PS. If you want the full list of libraries that I needed to install in order to compile correctly:
<code>
apt-get install gcc build-essential automake ncurses libncurses5-dev libtool termcap bison flex
</code>

Update: since I was using the stock ubuntu 10.04 server install apparmor was installed and keeping mysql from correctly installing it's databases in a non-standard directory. I kept getting this error:
<code>
/opt/xtradb/bin/mysql_install_db --user=mysql --ldata=/opt/xtradb/data
Installing MySQL system tables...
100802 10:26:54 [Warning] Can't create test file /opt/xtradb/data/ubuntu.lower-test
</code>

So just had to run a 
<code>
apt-get purge apparmor
</code>
restart and then run
<code>
/opt/xtradb/bin/mysql_install_db --user=mysql --ldata=/opt/xtradb/data
</code>
and finally I could start mysql
<code>
/opt/xtradb/bin/mysqld_safe --defaults-file=/opt/xtradb/conf/my.cnf --user=mysql --basedir=/opt/xtradb --datadir=/opt/xtradb/data --log-error=/var/log/xtradb/mysql.err &
</code>

