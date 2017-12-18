---
tags:
- jekyll
- examples
---

# Head

{{site.time}}

----

{% for stag in site.tags %}
{{ stag }}
{% endfor %}

----

{% for ptag in page.tags %}
{{ ptag }}
{% endfor %}

----

{% for tag in tags %}
{{ tag }}
{% endfor %}

----

{{tags}}

