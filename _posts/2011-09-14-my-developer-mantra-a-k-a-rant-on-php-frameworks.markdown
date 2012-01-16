--- 
wordpress_id: 144
layout: post
title: "My developer mantra a.k.a rant on php frameworks "
date: 2011-09-14 12:48:05 -04:00
wordpress_url: http://www.jessesanford.com/?p=144
---
Below is the reply I sent to the NYC PHP SIG mailing list on yet another "What are the top php frameworks" subject line... Read. Enjoy. Comment CRITICALLY and INTELLIGENTLY please.

<h1>"</h1>But if you are going to use someone else's code. make sure. Make for
damn sure... it has a wealth of CURRENT documentation and a lively
community.

My mantra... (a.k.a. rant)

I don't use software that tries to solve all problems. The worst
software grows out of goal-less development. Look for something with
clear, concise, decisive design decisions. The *nix moto... do one
thing and do it right. LESS CODE IS BETTER! A good example that will
hit close to home for some and I hope won't start a flame war is
Drupal. Until recently it was a bloated mess. It has started to clean
up it's act come version 7 but in my opinion it still does WAY too
much for it's own good.

I don't use software if the documentation is not up to snuff. Don't
take api/(insert language here)doc as enough. When you are getting up
to speed in a new technology you need much more handholding or face
hair loss. You need cookbooks, working examples, CURRENT api and
method signatures. etc.

I dont use software if it hasn't already been put into production some
very visible places. Use something you know works. Why be the first on
the block to see if X scales? You know how I know what scales. I have
seen larger stacks use it in the past. Safe bets are best.

I dont use software if it hasn't had at least 2 minor releases. Unless
you need a specific feature of a bleeding edge release just use what
you know will work! How do you know... because it's been around for a
while and people have used it and tested it.

I dont use software if it has no community involvement. (Don't
underestimate other peoples schedules... think about how busy you get
during your average work year. You may just have a project dropped
from under you. One core developer on an open source project is not
enough to bet the house on.)

I prefer my software to come with unit/functional tests. If nothing
else they can be used as examples. But the piece of mind you get as a
developer if you know what you are using has been tested thoroughly is
worth more than the most amazing untested functionality you can dream
of. Just think of all the times you banged your head on your desk
trying to figure out why something isn't doing what the documentation
said it was supposed to do. Only to find out yes... it is a bug in the
library you are using!

I hate using software. Less is better period. If you can get away
without using an extra daemon then do it. Don't add 5 more caching
layers and 3 more database abstractions just because they are the new
cool acronyms. It's only going to make your code less maintainable.
Over architecting and early optimization are what keep projects from
ever getting to launch.

The same thing goes for languages. Don't write your services in more
than 1 language if you can avoid it. If you are writing webapps from
the ground up limit yourself to one backend and one frontend language.

Don't add caching until you need it! The same thing goes for sharding
of your database schemas! KISS... across your entire stack. Not just
your code.

Whew... sorry about that. Back to the topic at hand.

Symfony 1.X is probably still more widely used than Symfony 2.x and
both are good. Although 2.x is a far more abstracted and multipurpose
framework 1.x is still great for the usual 3 tier webapps. Both are
very well documented.

CodeIgniter is a really great lightweight framework. If you are
looking for something straight forward, well documented and full of
examples this is also a good choice.

Zend had so much steam early on and then where did it go? I feel like
they have been sleeping.

Yii has been around for a while and same with Kohana but both have
less numbers than the above.

I have to revisit cake. I haven't used it in a while but for a while
it was seriously suffering on the documentation front.

If I was to pick a framework to run with on a new project I would
probably pick Symfony 2 because of it's forward thinking architecture.
It can seem overly complex so be warned. If you haven't already heard
of things like dependency injection or service containers you may feel
a little lost. The good thing is that there is a bunch of
documentation to get you up to speed. Also it makes use of useful
things like ESI and... HTTP... who knew!<h1>"</h1>
