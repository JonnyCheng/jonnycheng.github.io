---
layout: page
title: Articles
permalink: /article/
---

{% for category in site.categories %}
  <li class="category">{{ category | first }}
    <ul class="category-item">
    {% for posts in category %}
      {% for post in posts %}
        <li>
            <h2 class="list-post-title">
                <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
            </h2>
            <div class="list-post-date">
                <time>{{ post.date |date:"%Y-%m-%d %H:%M" }}</time>
            </div>
        </li>
      {% endfor %}
    {% endfor %}
    </ul>
  </li>
{% endfor %}