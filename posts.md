---
layout: page
title: News
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{site.url}}/{{ post.url }}">{{ post.title }}</a>
      <span style="margin-left:15px;font-style:italic;font-size:0.8rem">{{ post.date | date: "%Y-%m-%d" }}</span>
      <div style="margin-left:15px;font-style:italic">{{ post.excerpt }}</div>
    </li>
  {% endfor %}
</ul>
