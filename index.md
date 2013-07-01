---
layout: page
title: Jinliang's Blog
tagline: Supporting tagline
---
{% include JB/setup %}


## Update Author Attributes


    
## Sample Posts



<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


