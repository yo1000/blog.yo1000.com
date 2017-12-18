---
tags:
- jekyll
- examples
---

{{site.time}}

{% for tag in tags %}
{{ tag }}
{{ site.baseurl }}/tags#{{ tag | slugize }}
{% endfor %}
