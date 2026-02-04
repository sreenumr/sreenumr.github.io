---
layout: default
title: Home
---

<section class="max-w-3xl">
  <!-- <h1 class="text-5xl font-bold leading-tight">
    Hello World
  </h1> -->
<div class="flex flex-col ">
  <p class="mt-6 text-lg text-gray-600">
    I design, automate, and operate cloud-native systems.
    Focused on CI/CD, AWS, Kubernetes, and production-grade automation.
  </p>


<div class="text-lg">
  Projects
</div>
{% for project in site.data.projects %}
  {% include project-card.html project=project %}
{% endfor %}
</div>
</section>
