--- 
wordpress_id: 34
layout: post
title: Map-Reduce, Hadoop, Hadoop Streaming, Python and racism
date: 2009-11-27 14:25:08 -05:00
wordpress_url: http://www.jessesanford.com/?p=34
---
Here is my python script for grabbing the latest 1000 comments (the api only allows access to the latest 1000 unfortunately) and then checks them against a regexp for matching agains known racist words. Right now it is just looking for the N word. This script will be one of the inner MAP tasks in a series of Map-Reduce steps.

#!/usr/bin/env python

import sys

import gdata.youtube

import gdata.youtube.service

import re

racist_pattern = re.compile('.*igger.*', re.IGNORECASE)

#import pprint

#pp = pprint.PrettyPrinter(indent=4)

yt_service = gdata.youtube.service.YouTubeService()

#yt_service.developer_key = "" Â  Â  #turns out the developer key isn't necessary

urlpattern = 'http://gdata.youtube.com/feeds/api/videos/%s/comments?start-index=%d&amp;max-results=50'

for line in sys.stdin:

video_id = line.strip()

index = 1

url = urlpattern % (video_id, index)

#print url

comments = []

while url:

if index &lt; 20:

comment_feed = yt_service.GetYouTubeVideoCommentFeed(uri=url)

#comments.extend([ comment.content.text for comment in comment_feed.entry ])

for comment in comment_feed.entry:

if racist_pattern.match(comment.content.text):

print '%s\t%s\n' % (comment.author[0].name.text, comment.content.text)

#print [ 'Author: %s\t Comment: %s\n' % (comment.author[0].name.text, comment.content.text) for comment in comment_feed.entry ]

url = comment_feed.GetNextLink().href

index += 1

else:

#currently the google youtube gdata api will not support over 1000 comments

url = 'http://gdata.youtube.com/feeds/api/videos/%s/comments?start-index=951&amp;max-results=49' % video_id

comment_feed = yt_service.GetYouTubeVideoCommentFeed(uri=url)

for comment in comment_feed.entry:

if racist_pattern.match(comment.content.text):

print '%s\t%s\n' % (comment.author[0].name.text, comment.content.text)

#comments.extend([ comment.content.text for comment in comment_feed.entry ])

#print [ 'Author: %s\t Comment: %s\n' % (comment.author[0].name.text, comment.content.text) for comment in comment_feed.entry ]

break
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#!/usr/bin/env python</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">import sys</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">import gdata.youtube</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">import gdata.youtube.service</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">import re</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">racist_pattern = re.compile('.*igger.*', re.IGNORECASE)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#import pprint</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#pp = pprint.PrettyPrinter(indent=4)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">yt_service = gdata.youtube.service.YouTubeService()</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#yt_service.developer_key = "AI39si7MDdkK_3HKW7C-NykJxoCuBYSBk3GfFDdjEG7tHWmNIZKyLgnvLR9sj6D4wss3IXWQ-oIWm_hB29vb7oOFUCMk8OClMQ"</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">urlpattern = 'http://gdata.youtube.com/feeds/api/videos/%s/comments?start-index=%d&amp;max-results=50'</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">for line in sys.stdin:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">video_id = line.strip()</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">index = 1</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">url = urlpattern % (video_id, index)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#print url</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">comments = []</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">while url:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">if index &lt; 20:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">comment_feed = yt_service.GetYouTubeVideoCommentFeed(uri=url)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#comments.extend([ comment.content.text for comment in comment_feed.entry ])</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">for comment in comment_feed.entry:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">if racist_pattern.match(comment.content.text):</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">print '%s\t%s\n' % (comment.author[0].name.text, comment.content.text)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#print [ 'Author: %s\t Comment: %s\n' % (comment.author[0].name.text, comment.content.text) for comment in comment_feed.entry ]</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">url = comment_feed.GetNextLink().href</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">index += 1</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">else:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#currently the google youtube gdata api will not support over 1000 comments</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">url = 'http://gdata.youtube.com/feeds/api/videos/%s/comments?start-index=951&amp;max-results=49' % video_id</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">comment_feed = yt_service.GetYouTubeVideoCommentFeed(uri=url)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">for comment in comment_feed.entry:</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">if racist_pattern.match(comment.content.text):</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">print '%s\t%s\n' % (comment.author[0].name.text, comment.content.text)</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#comments.extend([ comment.content.text for comment in comment_feed.entry ])</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">#print [ 'Author: %s\t Comment: %s\n' % (comment.author[0].name.text, comment.content.text) for comment in comment_feed.entry ]</div>
<div id="_mcePaste" style="position: absolute; left: -10000px; top: 0px; width: 1px; height: 1px; overflow-x: hidden; overflow-y: hidden;">bre</div>
