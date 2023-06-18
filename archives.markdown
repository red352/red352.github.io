---
layout: page
title: 归档
permalink: /archives/
---

<div class="archive-container">
  

  {% assign sorted_posts = site.posts | sort: 'date' | reverse %}
  {% assign currentyear = "" %}
  {% assign currentmonth = "" %}

  {% for post in sorted_posts %}
    {% capture postyear %}{{ post.date | date: "%Y" }}{% endcapture %}
    {% capture postmonth %}{{ post.date | date: "%m" }}{% endcapture %}
    {% capture postday %}{{ post.date | date: "%d" }}{% endcapture %}

    {% if postyear != currentyear %}
      {% if currentyear != "" %}
        </ul>
      {% endif %}
      <h2>{{ postyear }}</h2>
      {% assign currentyear = postyear %}
      {% assign currentmonth = "" %}
      <ul>
    {% endif %}

    {% if postmonth != currentmonth %}
      {% if currentmonth != "" %}
        </ul>
      {% endif %}
      <h3>{{ post.date | date: "%B" }}</h3>
      {% assign currentmonth = postmonth %}
      <ul>
    {% endif %}

    <li><a href="{{ post.url }}">{{ post.date | date: "%B %d, %Y" }} - {{ post.title }}</a></li>
    
    {% if forloop.last %}
      </ul>
    {% endif %}
    
  {% endfor %}

