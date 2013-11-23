---
layout: page
title: Patrick Hayes
---
{% include JB/setup %}

My name is Patrick Hayes. I live in New York City, and I'm a software engineer at Foursquare.

There will be a blog here someday.

{% if site.posts %}
## Posts
  <ul class="posts">
    {% for post in site.posts %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endif %}

