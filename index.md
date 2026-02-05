---
layout: default
title: Home
---

## I design, automate, and operate cloud-native systems.
## Focused on CI/CD, AWS, Kubernetes, and production-grade automation.

### Projects

<div markdown="0">
{% for project in site.data.projects %}
  {% include project-card.html project=project %}
{% endfor %}
</div>
