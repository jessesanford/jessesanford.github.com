--- 
wordpress_id: 8
layout: post
title: When good code goes BAD or The Unfortunate Misuse of Good Software Development Principles
excerpt: I had a bit of a quandary today as I reviewed some code today and I decided to write up this little post on the experience. In essence it came down to choosing between code clarity and code reuse. I am a huge fan of both and they almost never conflict in fact they almost always exist in a symbiotic state. But not this time.
date: 2009-11-09 22:29:31 -05:00
wordpress_url: http://www.jessesanford.com/?p=8
---
I had a bit of a quandary today as I reviewed some code today and I decided to write up this little post on the experience. In essence it came down to choosing between <a href="http://www.fulltablescan.com/index.php?/archives/140-Toms-First-Law-Code-Readability-is-the-Key-to-Long-term-Software-Quality.html">code</a> <a href="http://haacked.com/archive/2007/04/20/write-readable-code-by-making-its-intentions-clear.aspx">readability</a> and <a href="http://en.wikipedia.org/wiki/Code_reuse">code</a> <a href="http://devlicio.us/blogs/tim_barcz/archive/2009/03/09/real-life-code-reuse.aspx">reuse</a>. I am a huge fan of both and they almost never conflict in fact they almost always exist in a symbiotic state. But not this time.

This title of this post has been buzzing around in my head recently as I have been doing a lot of performance tuning on a very large and HIGHLY trafficked web site of a major print publication. Anyway I was recently reading (reference coming soon! until then dig through my delicious links) a blog post on the topic of performance tuning and why it is silly how often people micro-tuning. Think of that in the same negative respect that you think of micro-managing. It's a waste of time and resources. The moral: even though performance tuning is normally a good thing, after a while it looses it's value. You spend your money on the low hanging fruit (AHEM more hardware!)

Back to today. The code that I was reviewing had a few very cryptic stanzas that included a call to a function that consumed a single integer based parameter which after thorough inspection simply ended up determining if the query it finally triggers against the database is ascending or descending. Now I honestly doubt this developer (no matter how Jr) actually thought they would be increasing the performance of the application by including this integer parameter. In fact I am sure that they assumed that they were doing a good thing by reusing code (which I am a HUGE proponent of normally!) but when it came time for another developer to get into the code to augment it slightly it took exponentially more time then it would have if the original developer had simply a) created the database queries inline or b) created two separate nearly identical functions. Instead (fyi we are using an <a href="http://en.wikipedia.org/wiki/Object-relational_mapping">ORM</a> that follows the <a href="http://en.wikipedia.org/wiki/Active_record_pattern">ActiveRecord</a> <a href="http://martinfowler.com/eaaCatalog/activeRecord.html">Pattern</a>) the object has one single method <code>getNextArticle(int foo)</code> that returns both the previous AND the next article! How confusing! Anyway the best solution that required the least new untested code at this point was to wrap that function with two new functions that map to the integer values. Now we have:

<code>
/*Note that the following function has a different method signature than the original and thus can coexist due to the <a href="http://en.wikipedia.org/wiki/Method_overloading">method overloading</a> feature of java*/
getNextArticle(){ 
 return this.getNextArticle(1);
}
</code>

<code>
getPreviousArticle(){
  return this.getNextArticle(2);
}
</code>

I can already hear someone out there saying: "That's silly why not re-factor the whole object so that it has:

<code>getNextArticle()</code> 

<code>getPreviousArticle()</code>

wrapping some new function that is called with a String paramater like ASC or DESC." Well to put it simply, time. Regression testing takes time and we would have to do a heck of a lot more of it on this project that unfortunately does not yet have 100% unit test coverage (to my DISDAIN) and does not have an automated testing setup. So the moral of the story is this. Even though code reuse is almost always a GOOD sign of proper software development practices sometimes it can lead to poor readability which then becomes more of a maintenance problem than the code reuse actually solves. In retrospect it would have been great to have found this code before it went through thorough user acceptance testing. However due to the accelerated nature of meeting client demands and working on unrealistic project schedules we were not able to do enough peer <a href="http://en.wikipedia.org/wiki/Code_review">code reviews</a> of code nor were we able to create a team large enough to do <a href="http://en.wikipedia.org/wiki/Pair_programming">pair programming</a>. 

AHH how the <em>real world</em> always ruins principles, techniques, patterns and paradigms that work so well on paper!
