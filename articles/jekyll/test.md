---
tags:
- jekyll
- examples
---

{{site.time}}

{% for tag in tags %}
<a href="{{ site.baseurl }}/tags#{{ tag | slugize }}">
  {{ tag }}
</a>
{% endfor %}
