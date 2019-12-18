---
layout: archive
permalink: /articles/
title: &title "Articles"
pagination: 
  enabled: true
  category: articles
---

{% for post in site.categories.articles %}
  {% include entry.html %}
{% endfor %}