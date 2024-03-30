---
layout: page
title: 归档
subtitle: 按时间倒序
permalink: /archives/
---

<div class="archive-container">
  {% assign postsByYearMonth = site.posts | group_by_exp:"post", "post.date | date: '%Y-%m'" %}
  
  {% for yearMonth in postsByYearMonth %}
    <div class="archive-section">
      <h2 class="archive-heading">{{ yearMonth.name }}</h2>
      <ul class="archive-list">
        {% for post in yearMonth.items %}
          <li class="archive-item">
            <span class="archive-date">{{ post.date | date: "%Y-%m-%d" }}</span>
            <a class="archive-link" href="{{ post.url }}">{{ post.title }}</a>
          </li>
        {% endfor %}
      </ul>
    </div>
  {% endfor %}
  
  <div class="archive-navigation">
    {% if page.previous %}
      <a class="archive-nav-link" href="{{ page.previous.url }}">上一页</a>
    {% endif %}
    
    {% if page.next %}
      <a class="archive-nav-link" href="{{ page.next.url }}">下一页</a>
    {% endif %}
  </div>
</div>