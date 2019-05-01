---
layout: archive
permalink: /ctf/
title: &title "CTF Writeup"
excerpt: &excerpt "Some stuff about CTF."
introduction: *excerpt
pagination: 
  enabled: true
  category: ctf
---

{% for post in site.categories.ctf %}
  {% include entry.html %}
{% endfor %}