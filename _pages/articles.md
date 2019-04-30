---
layout: archive
permalink: /articles/
title: &title "Articles"
alt_title: *title
excerpt: &excerpt "A collection of thoughts, inspiration, mistakes, and other long-form minutia I've written. For smaller, more regular tidbits --- peruse the [notes section](/notes/)."
introduction: *excerpt
pagination: 
  enabled: true
  category: articles
---

{% for post in site.articles %}
    {% include post-list.html %}
{% endfor %}