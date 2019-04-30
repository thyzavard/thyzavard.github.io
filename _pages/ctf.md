---
layout: archive
permalink: /ctf/
title: &title "CTF Writeup"
alt_title: *title
excerpt: &excerpt "A collection of thoughts, inspiration, mistakes, and other long-form minutia I've written. For smaller, more regular tidbits --- peruse the [notes section](/notes/)."
introduction: *excerpt
pagination: 
  enabled: true
  category: ctf
---

{% for post in site.ctf %}
    {% include post-list.html %}
{% endfor %}