---
layout: page
title: Theory & Notes
icon: fas fa-book-open
order: 3
---

<p class="theory-subtitle">Here I document my conceptual understanding, literature reviews, and mathematical notes on control systems, robotics, and learning.</p>

<div class="theory-list mt-4">
  {% for post in site.categories.Theory %}
    <a href="{{ post.url | relative_url }}" class="theory-entry">
      <span class="theory-entry-title">{{ post.title }}</span>
      <span class="theory-entry-date" style="font-family: monospace; font-size: 0.8rem; color: #6deaef;">
        {{ post.date | date: "%b %d, %Y" }}
      </span>
    </a>
  {% endfor %}
</div>