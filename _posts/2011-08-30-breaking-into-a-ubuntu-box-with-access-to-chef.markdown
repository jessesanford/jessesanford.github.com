--- 
wordpress_id: 110
layout: post
title: Breaking into a ubuntu box with access to chef
date: 2011-08-30 11:03:42 -04:00
wordpress_url: http://www.jessesanford.com/?p=110
---
You would think this would be easier than it was. The Chef daemon that Rightscale runs on the ec2 instances it manages runs as root. However I had a few requirements. A) I had to do it all from my galaxy tab. B) I had to use only the Rightscale admin pannel. C) I could only write my commands in bash (I think this is all the rightscripts support) D) The bash commands I wrote were all executed by chef in a forked process so normal *nix piping and such wouldn't work. E) I didn't want to open up the root user for direct login.

SO to start I had to create a new user, give him a home directory and add him to the admin group.

Then I had to put the galaxy tab's public key into the user's authorized_keys file.

Then I had to allow for passwordless sudo for that user.

The last item on the list was simply because I could not find a way of having chef in it's execution of my bash scripts to pipe a password into passwd. Passwd even when run as root expects a password to be entered in on the tty. So of course I attempted all kinds of <code>echo "password" | passwd jesse</code> and <code>passwd jesse < echo "password"</code> combos. None worked. I asssume because the pipes were being misdirected by chef in some way. If someone could explain that to me I would be grateful!

Also creating the authorized_keys file proved very tricky. Since it didn't already exist I had to create a new file from scratch. I started by simply touching the file <code>touch /home/jesse/.ssh/authorized_keys</code> and then trying to <code>echo "pubkeymaterial" >> /home/jesse/.ssh/authorized_keys</code> but again because of the piping issue explained above nothing would work. I kept ending up with an empty file.

Sed to the rescue!
I broke out sed and started by simply trying to add a line to the file: <code>sed -i -e "1a\\
pubkeymaterial" /home/jesse/.ssh/authorized_keys</code>

Again blank file!

So then I tried to replace the empty line...

<code>sed -i -e "s/^$/pubkeymaterial" /home/jesse/.ssh/authorized_keys</code>

Again blank file!

Turns out sed needs SOMETHING in the file before you can insert add or replace text within it.

So I had to figure out how to get some data into the authorized keys file without an interactive console (no vi or emacs sorry!) AND without pipes (no echo redirection sorry!)

DD to the rescue!
So how do you get some data into a file? Read it from somewhere! Where? well there are of course a few std places you could "mv/cp" a file from (I had thought about moving the .bash_history into place and then deleting everything but the first line. I assume that would have worked fine but would it be TRULY portable?)

Instead I decide to just put a zero into place.

<code>dd if=/dev/zero of=/home/jesse/.ssh/authorized_keys bs=1 count=1</code>

Now sed would work!

<code>sed -i -e "s/^.*/pubkeymaterial" /home/jesse/.ssh/authorized_keys</code>

Back to my terminal on the galaxy tab I was now able to login as jesse interactively. BUT because my password had never been set I couldn't actually sudo. 

<code>sudo ls ~/ </code> would hange asking me for a password that I didn't have.

SO I need sudo to allow my Jesse user to sudo without prompting me for a password.

Back to sed.

sh -c "/bin/sed -i -e '5a\\
Defaults:jesse !authenticate' /etc/sudoers"

Wallah! now back in the gallaxy tab's terminal I could <code>sudo ls ~/</code> to my hearts content.

I finished up interactively in the gallaxy tab terminal. setting a password for myself.

<code>sudo passwd jesse</code> and then removing the passwordless sudo Default from the /etc/sudoers file.

Also it should be mentioned that I created my user within the admin group from the start using the following.

<code>adduser --ingroup admin --home /home/jesse/ --shell /bin/bash jesse<code>

And of course if you caught it I did have to create the .ssh directory and own it.

<code>mkdir /home/jesse/.ssh/</code>

<code>chown -R /home/jesse/.ssh</code>

All in all it took a few hours. A hell of a lot longer than it would have if I had simply cheated and added my key directory the root user and then allowed root login but that would have been against the ubuntu way and a heck of lot less interesting :)
