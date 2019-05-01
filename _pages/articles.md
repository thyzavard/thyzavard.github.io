---
layout: archive
permalink: /articles/
title: &title "Articles"
excerpt: &excerpt "A collection of thoughts, inspiration, mistakes, and other long-form minutia I've written. For smaller, more regular tidbits --- peruse the [notes section](/notes/)."
introduction: *excerpt
pagination: 
  enabled: true
  category: articles
---

{% for post in site.categories.articles %}
  {% include entry.html %}
{% endfor %}