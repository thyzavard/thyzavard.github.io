---
permalink: /notes/
title: "All my notes"
excerpt: "A collection of thoughts, inspiration, mistakes, and other long-form minutia I've written. For smaller, more regular tidbits --- peruse the [notes section](/notes/)."
pagination: 
  enabled: true
  category: notes
---

{% for post in site.categories.notes %}
  {% include archive-single.html %}
{% endfor %}
