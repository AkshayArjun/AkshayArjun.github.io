---
title: Theory & Notes
icon: fas fa-book-open
order: 3
---

Here I document my conceptual understanding, literature reviews, and mathematical notes on control systems, robotics, and learning.

<ul>
{% for post in site.categories.Theory %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    - <i>{{ post.date | date: "%B %d, %Y" }}</i>
  </li>
{% endfor %}
</ul>