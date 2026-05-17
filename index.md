---
layout: default
title: Home
---

## Welcome to Practice QA

Your resource for software testing and automation interview questions.

## Latest Interview Questions

<ul>
{% for post in site.posts limit:30 %}
  <li style="margin: 10px 0;">
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    <span style="color: #666; font-size: 0.9em;">({{ post.date | date: "%Y-%m-%d" }})</span>
  </li>
{% endfor %}
</ul>

## Total Posts

Total interview questions: **{{ site.posts.size }}**
