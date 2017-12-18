---
tags:
- jekyll
- examples
---

{% for tag in site.tags %}
<em>{{ tag }}</em>
<a href="{{ site.baseurl }}/tags#{{ tag | slugize }}">{{ tag }}</a>
{% endfor %}
