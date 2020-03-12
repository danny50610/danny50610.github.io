---
layout: page
title: Misc
permalink: /misc/
---

一些雜項頁面

{% for misc_page in site.misc %}
  <a href="{{ misc_page.url }}">{{ misc_page.title | escape }}</a>：{{ misc_page.description | escape }}
{% endfor %}
