---
layout: page
title: Posts
permalink: /posts/
header: true

---
<p>{{ site.title }}</p>
<ul>
{% for post in site.posts %}
  <li>
    {{ post.date | date: "%Y-%m-%d"  }} &mdash; <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>
