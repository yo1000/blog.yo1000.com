---
tags:
- jekyll
- examples
---

{% for tag in site.tags %}
<a href="{{ site.baseurl }}/tags#{{ tag | slugize }}"> {{ tag }} </a>
{% endfor %}
