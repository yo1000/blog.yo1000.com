---
tags:
- jekyll
- examples
---

{% for tag in site.tags %}
<a href="{{ site.baseurl }}/tags#{{ tag }}"> {{ tag }} </a>
{% endfor %}
