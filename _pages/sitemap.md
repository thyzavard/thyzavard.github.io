---
permalink: /sitemap/
title: "Sitemap"
excerpt: "An index of all the pages found on blog.yzavard.fr"
---

Simple sitemap with all sections and pages of the site. You can also find an [XML version](/sitemap.xml).

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