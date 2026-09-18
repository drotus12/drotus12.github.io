---
icon: fas fa-book
order: 5
---

<ul>
{% for article in site.articles %}
  <li><a href="{{ article.url }}">{{ article.title }}</a></li>
{% endfor %}
</ul>