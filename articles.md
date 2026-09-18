---
title: 記事一覧
permalink: /articles/
layout: page
---

<ul>
{% for article in site.articles %}
  <li>
    <a href="{{ article.url }}">{{ article.title }}</a>
  </li>
{% endfor %}
</ul>
