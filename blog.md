---
layout: default
title: Blog
---

<div class="space-y-6">

  <header>
    <h1 class="text-2xl font-medium">Blog</h1>
    <p class="text-sm text-gray-500">
      Technical notes and postmortems.
    </p>
  </header>

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
