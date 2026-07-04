---
layout: default
title: Arild Eide
---

# Welcome

This is my blog. It has no value for anyone else than myself.


## Posts

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <small>{{ post.date | date: "%Y-%m-%d" }}</small>
  </li>
{% endfor %}
</ul>
