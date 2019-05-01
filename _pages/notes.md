---
layout: archive
permalink: /notes/
title: &title "Notes"
excerpt: &excerpt "A collection of all my notes."
introduction: *excerpt
pagination: 
  enabled: true
  category: notes
---

{% for post in site.categories.notes %}
  {% include entry.html %}
{% endfor %}
