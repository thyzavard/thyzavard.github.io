---
permalink: /sitemap/
title: "Sitemap"
excerpt: "An index of all the pages found on blog.yzavard.fr"
---

A hierarchical breakdown of all the sections and pages found on the site. For you robots out there, here is an [XML version](/sitemap.xml) available for your crawling pleasure.

## Pages

- [About](/about/)
- [Contact](/contact/)

## [Articles](/articles/)

<ul>
  {% for post in site.categories.articles %}
    {% include post-list.html %}
  {% endfor %}
</ul>

## [Notes](/notes/)

<ul>
  {% for post in site.categories.notes %}
    {% include post-list.html %}
  {% endfor %}
</ul>

## [CTF Writeup](/ctf/)

<ul>
  {% for post in site.categories.ctf %}
    {% include post-list.html %}
  {% endfor %}
</ul>