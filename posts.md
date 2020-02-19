---
layout: page
title: Posts
permalink: /posts/
header: true

---

<p>everything &middot; <a href="/article-reviews/">no notes</a></p>

<ul>
{% for post in site.posts %}
  <li>
    {{ post.date | date: "%Y-%m-%d"  }} &mdash; <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>
