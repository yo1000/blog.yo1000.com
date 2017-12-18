---
tags:
- jekyll
- examples
---

{{site.time}}

----

{% for tag in site.tags %}
{{ tag }}
{% endfor %}

----

{% for tag in page.tags %}
{{ tag }}
{% endfor %}

----

{% for tag in tags %}
{{ tag }}
{% endfor %}

----

{{tags}}

