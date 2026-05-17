---
layout: home
title: Practice QA
---

Welcome to **Practice QA** - your resource for software testing and automation interview questions.

## Latest Posts

<ul>
{% for post in site.posts limit:30 %}
  <li>
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    <small>({{ post.date | date: "%Y-%m-%d" }})</small>
  </li>
{% endfor %}
</ul>

Total posts: {{ site.posts.size }}
