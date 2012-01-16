--- 
wordpress_id: 39
layout: post
title: Why Drupal polls don't play well with full page caching or how a 3-tier architecture works.
date: 2009-12-17 11:56:25 -05:00
wordpress_url: http://www.jessesanford.com/?p=39
---
I had a client who didn't understand why caching was breaking things on their site. Of particular nuisance was the polls. To be fair, grasping why something as easy as polls works without caching and then does not work with caching can be confusing. Here is the email I wrote to help educate all of us. Below you will see the contents of my email to them.

&lt;snip&gt;

To begin I think it is necessary to explain why things are cached and why caching has an impact on editorial workflow and functionality. Specifically this is in reference to polls. It's going to take a bit to grasp what is happening so please read slow and bare with me.

Full web page caching or HTTP acceleration is done in part to effectively reduce the load experienced by an application and also to mask performance bottlenecks in the application to users.

Without caching, every time a user makes a request, that request eventually has some action that is performed by business logic in the application and ultimately the database. That action can actually be one or THOUSANDS of connections and queries. Those thousands of queries per user request are normally NOT unique as in one request will normally cause the same exact connections and queries to be performed as the next request for the same content. So as the number of users goes up the number of requests go up and ultimately the number of connections and queries on the db will go up (however far more dramatically).

Here are some scenarios:

1 user makes 1 request for 1 page that takes 500 queries to render = 500 queries on the database.

1 user makes 3 requests for 3 different pages each taking 500 queries to render = 1500 queries on the database.

2 users each make 3 requests for 3 different pages each taking 500 queries to render = 3000 queries.

It's linear when you look at it in this fashion but if you think about how the database actually performs transactions you will see that the performance hit becomes EXPONENTIAL.

The database performs each transaction in a fashion that is sometimes blocking only allowing a single transaction to request a certain record at a time (rare in read only scenarios) OR the io needed to service requests can actually cause the processor to wait while the hard disks are saturated (common on high volume, data intensive applications). This wait time experienced by the processor can quickly add up to minutes experienced by an individual request. Minutes can become tens of minutes even hours as queries stack up waiting for the hard disks to return the data to the queries in the queue ahead of them.

There are many ways to solve this problem but the first is always to remove the waste.

Why perform all those non-unique calculations over and over again when you can just do them once, store the results and then return those stored results when the same non-unique request comes back again?

That is caching.

There are many levels of caching.

We can cache the query results for each query performed on the database. Every single one of the thousands of unique queries is currently kept cached automatically by the datbase for a certain period of time until it is thrown away to make room for "Hotter" queries.

We can also cache the results of the processing of the business logic (in the case of web applications this is usual some sort of html or xml output) those results can be used to produce an actual .html file on the hard disk.

This called "page level" caching.

<strong>Hotness explained </strong>(this is not unique to databases this is for most simple caching algorithms)

If a certain request is made for a certain piece of data lets assign it an importance value of 1.

If a subsequent request is made for that same data lets add 1 to that importance value so that it is now 2

If no other request is made for that data it's value stays at 2 for some period of time.

Now let's graph this importance or hotness value:

The x axis is some deliniation of the volume of data. Lets just presume that all your data is in a single directory and it all has random alphabetical filenames. Then X would possibly start at at AKURALEJD and end at ZNFNALHE. NOTE the filenames are RANDOMLY assigned to the data.

The Y axis is the hotness or importance value assigned to each piece of data so the Hotter the data the higher it is plotted on the Y axis.

What you will find is that SOME portion of the data is hotter than others. The plot of said hotness follows a bell curve.

So why not just cache ALL possible data ever produced by an application?

Memory is limited. We can't possibly account for every single unique view of the data. SO we can only keep a portion of that data in the cache. There for we draw vertical lines on our graph starting at the middle of the bell curve and expanding them until the content contained in distance between them is equal to amount of available memory. After that the "tail ends" remain un-cached.

<img class="alignnone size-full wp-image-43" title="ttl_cache_bell_curve1-300x210" src="http://www.jessesanford.com/wp-content/uploads/2009/12/ttl_cache_bell_curve1-300x210.png" alt="ttl_cache_bell_curve1-300x210" width="300" height="210" />

Now what happens when what's hot changes? Well we have to make room in the cache so that this new data can be put in. How do we know what to throw out of the cache to make room? Well we introduce a lifetime or a TTL (Time To Live) on each element in the cache.

So in it's most basic form a TTL will tell the cache to "Throw out anything older than X"

That means that as time goes on if something HAS not been requested in a longer period of time than the TTL is set to then it will eventually make it's way out of the cache.

This aging process also allows for the content that is cached to be "refreshed" or updated periodically. SO if you add NEW data or change data in the application then those changes will eventually make their way into the cache and ultimately be seen by people making requests for cached data.

Ok so that all sounds good so why all the issues with caching?

Before we get into this we need to take an even farther step back. Let's look at how our content is uniquely named.

When it comes to the web our content is uniquely named by it's URL (uniform resource locator) See: <a href="http://en.wikipedia.org/wiki/Uniform_Resource_Locator">http://en.wikipedia.org/wiki/Uniform_Resource_Locator</a>

With a unique URL the cache is able to determine "what is what" or more technically what the "STATE" of the application is for that URL. <em>The web used to be so simple.</em>

What webservers do with URL's:
I will briefly explain the two most common functions a web server performs. These are:

Responding to GET requests. (commonly associated with clicking a link)

Responding to POST requests. (commonly associated with pushing a submit button)

NOTE that BOTH are "REQUESTS" there for if you click a link OR click a submit button you are "REQUESTING" something from the webserver. In the case of a POST request most times you are also sending something extra to the webserver (for instance a contact form OR your answers to a poll!)

Once the webserver receives your request it will check to see if it is a GET or a POST, then it will check to see what file was requested and then if the file is of a certain type it will sometimes offload the request to some application server (for instance PHP running as a cgi process). This application server (PHP for instance) will then process the request (taking into account the information in the GET or POST, making needed database queries etc.) finally the application server returns a response back to the webserver which will in turn return the response to your browser.

That response may or may return unique information depending on what was passed in the GET or POST request to the webserver (and ultimately to the application server) NOTE: In the past ALL requests both GET and POST caused the browser to refresh the page. <em>THAT fact alone is what defined WEB 1.0.</em>

Enter AJAX.

Now web 2.0. We no longer have Unique URLS for every piece of content. More importantly we don't have page refreshes upon every request. The same URL/page might look one way when you first visit it and then let's just say (for the sake of a timely example) you submit a poll for instance. If the submission of that poll does not navigate you away from the page to a new URL then with page level caching you will never see the results of the poll as your submission will only return the same exact cached page that you first saw since the url is the same. THEREFOR you will see the UNSUBMITTED poll once again.

So then how do we deal with caching AND AJAX?

We do so in one of two ways.

1) By maintaining STATE or at least some subset of the state on the client side (within the browser's cookies).

OR

2) By caching only small parts of a page that don't change and allowing the other parts of a page to remain UNCACHED. This is commonly called "partial caching".

So why not use partial caching on my project?

Partial caching is entirely application dependent. It has to be architected into the application from the start. You unfortunately cannot "just add" partial caching because someone has to take the time to determine which portions of an application CAN be cached and which portions CANNOT. Most times off the shelf web applications cannot have partial caching added on top of them. Most traditional CMS applications and legacy web applications are not Web 2.0 "savy" enough.

Drupal falls into a gray area. Some modules are smart enough. Others are not. In general you cannot use partial caching with drupal without some legwork.

SO we use the page level caching described above. Now what?

Well now we have to take into account the functionality implications of a page looking the same way no matter what every time you visit it. Sounds like it doesn't fix anything, and it doesn't, without some tricks.

To start we can tell the cache to NEVER respond to a POST request with cached content. That means that every time someone submits a poll then show them the results. The results are generated each time by the application server and then the unique response is sent back to the browser.

Why does the poll still NOT work on the site then?

The long and the short of it:

It does. BUT unfortunately the poll is one of those modules that was NOT written to be used with page level caching.

What's happening is this:

The first time you visit the site (or at least before ever taking the poll) you get there by typing www.yourpage.com into your browser. That is a GET request and the homepage is retrieved from the cache. You see the poll and you ARE allowed to vote and you DO see your results. Your request is sent via a POST to the application servers and your response is returned by the application servers... NOT the cache because it was a POST request.

Then you navigate away from that page and everything is fine. Then let's say you come back to the home page by clicking the logo in the top left. That is a GET request. The homepage is returned FROM THE CACHE this time and you do NOT see your results in the poll. You see a poll that has not been taken yet. Because that is how the homepage was cached originally.

SO then you try to take the poll again and this time it doesn't let you submit. Why? Because the poll module in drupal was built to only allow ONE vote per person per poll. That means that you CANT submit the poll more than once.

So since you have already submitted and since the poll is cached as unsubmitted you are stuck with an unsubmittable poll.

DARN! How do we fix this?

The easy way: Make the polls allow more than one vote per person per poll. Really why is that so bad?

The hard way: Rewrite the poll module correctly to refresh itself via another Ajax call for an un-cached version of the results. This means essentially writing a new poll module.

I hope that explains why the polls are not working. Maybe you picked up a thing or two a long the way.

&lt;/snip&gt;
