---
layout: page
title: Article reviews
permalink: /article-reviews/
header: true

---

<p><a href="/posts/">everything</a> &middot; no notes</p>

<ul>
{% for post in site.posts %}
    { % if post.tag == "review" %}
    <li>
        {{ post.date | date: "%Y-%m-%d"  }} &mdash; <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endif %}
{% endfor %}
</ul>
