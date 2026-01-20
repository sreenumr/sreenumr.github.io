---
layout: page
title: Blog
---

{% for post in site.posts %}
### [{{ post.title }}]({{ post.url }})

{{ post.excerpt }}

<small>
{{ post.date | date: "%b %d, %Y" }}
{% if post.tags %} • {{ post.tags | join: ", " }}{% endif %}
</small>

---
{% endfor %}
