---
layout: page
title: 分类
permalink: /categories/
---

<ul>
{% for category in site.categories %}
  <li>
    <h2>{{ category | first }}</h2>
    <ul>
      {% for post in category.last %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    </ul>
  </li>
{% endfor %}
</ul>