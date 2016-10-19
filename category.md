---
layout: page
title: 文章分类
permalink: /category/
---

<ul class="tag-box inline">
{% assign tags_list = site.categories %}  
  {% if tags_list.first[0] == null %}
    {% for tag in tags_list %} 
      <li>#<a href="#{{ tag }}">{{ tag }} <span>{{ site.tags[tag].size }}</span></a></li>
    {% endfor %}
  {% else %}
    {% for tag in tags_list %} 
      <li>#<a href="#{{ tag[0] }}">{{ tag[0] }} <span>{{ tag[1].size }}</span></a></li>
    {% endfor %}
  {% endif %}
{% assign tags_list = nil %}
</ul>

{% for tag in site.categories %} 
  <h4 id="{{ tag[0] }}">#{{ tag[0] }}</h4>
  <ul class="post-list">
    {% assign pages_list = tag[1] %}  
    {% for post in pages_list %}
      {% if post.title != null %}
      {% if group == null or group == post.group %}
      <li><h5 class="category-list-post-title">
                <time class="category-list-post-time">{{ post.date |date:"%Y-%m-%d" }}</time> <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a> 
            </h5></li>
      {% endif %}
      {% endif %}
    {% endfor %}
    {% assign pages_list = nil %}
    {% assign group = nil %}
  </ul>
{% endfor %}