---
permalink: /ctf/
title: "All my CTF Writeup"
excerpt: "A collection of thoughts, inspiration, mistakes, and other long-form minutia I've written. For smaller, more regular tidbits --- peruse the [notes section](/notes/)."
pagination: 
  enabled: true
  category: ctf
---

{% for post in site.categories.ctf %}
  {% include archive-single.html %}
{% endfor %}