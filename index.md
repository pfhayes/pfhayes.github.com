---
layout: page
title: Patrick Hayes
---
{% include JB/setup %}

Hi there. My name is Patrick Hayes.
I live in New York City, and I'm a software engineer at Foursquare.
This is a blog where you can read blog posts.

<hr/>

### Recent Posts
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<hr/>

{% assign post = site.posts | first %}
## Latest Post: {{ post.title }}
{{ post.content }}


