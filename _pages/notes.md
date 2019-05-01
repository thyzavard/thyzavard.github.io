---
layout: archive
permalink: /notes/
title: &title "Notes"
excerpt: &excerpt "A collection of thoughts, inspiration, mistakes, and other long-form minutia I've written. For smaller, more regular tidbits --- peruse the [notes section](/notes/)."
introduction: *excerpt
pagination: 
  enabled: true
  category: notes
---

{% for post in site.categories.notes %}
  {% include entry.html %}
{% endfor %}
