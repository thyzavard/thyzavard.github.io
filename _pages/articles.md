---
layout: archive
permalink: /articles/
title: &title "Articles"
excerpt: &excerpt "A list of all my articles."
introduction: *excerpt
pagination: 
  enabled: true
  category: articles
---

{% for post in site.categories.articles %}
  {% include entry.html %}
{% endfor %}