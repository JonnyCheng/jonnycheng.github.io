---
layout: page
title: Category
permalink: /category/
---

{% for category in site.categories %}
  <li class="category">#{{ category | first }}
    <ul class="category-item">
    {% for posts in category %}
      {% for post in posts %}
        <li>
            <h3 class="category-list-post-title">
                <time class="category-list-post-time">{{ post.date |date:"%Y-%m-%d" }}</time> <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a> 
            </h3>
        </li>
      {% endfor %}
    {% endfor %}
    </ul>
  </li>
{% endfor %}