---
layout: page
title: 分类
subtitle: 按A-Z排序
permalink: /categories/
---

{% assign sortedCategories = site.categories | sort %}
<div class="category-container">
  {% for category in sortedCategories %}
    <div class="category-card">
      <div class="category">
        <h2>{{ category | first }}</h2>
        <ul>
          {% for post in category.last %}
            <li><a href="{{ post.url }}">{{ post.title }}</a></li>
          {% endfor %}
        </ul>
      </div>
    </div>
  {% endfor %}
</div>