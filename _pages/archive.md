---
layout: archive
title: All Posts
permalink: /archive/
---

{% include base_path %}

{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}