---
layout: default
title: Blog
---

<div class="space-y-6">

  <section class="space-y-4">
    {% for post in site.posts %}
      <div>
        <a href="{{ post.url }}" class="text-blue-600 hover:underline">
          {{ post.title }}
        </a>
        <p class="text-xs text-gray-500">
          {{ post.date | date: "%d %b %Y" }}
        </p>
      </div>
    {% endfor %}
  </section>

</div>
