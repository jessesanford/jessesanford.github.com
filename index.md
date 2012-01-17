---
layout: page
title: Jesse Sanford - Durrability, Scalability, Availability
header: Durrability, Scalability, Availability
---

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

