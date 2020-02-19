---
layout: page
title: Reviews
permalink: /reviews/
header: true

---

<ul>
{% for post in site.posts %}
  {% if post.tag %}
    <li>
      {{ post.date | date: "%Y-%m-%d"  }} &mdash; <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endif %}
{% endfor %}
</ul>
