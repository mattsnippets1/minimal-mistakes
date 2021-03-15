---
layout: archive
permalink: /archive/
---

{% include base_path %}

{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}
