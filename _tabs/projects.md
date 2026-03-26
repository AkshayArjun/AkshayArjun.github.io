---
layout: page
title: Projects
icon: fas fa-robot
order: 2
---

<p class="proj-subtitle">A log of the autonomous systems, control frameworks, and robotics hardware I have built.</p>

<div class="proj-grid">
{% for post in site.categories.Projects %}
<div class="proj-card" onclick="window.location='{{ post.url | relative_url }}'" style="cursor:pointer;">
{% if post.image %}<img src="{{ post.image.path | relative_url }}" alt="{{ post.title }}" class="proj-card-img">{% endif %}
<div class="proj-card-overlay"></div>
<div class="proj-card-body">
<div class="proj-card-title">{{ post.title }}</div>
<div class="proj-card-excerpt">{{ post.content | strip_html | truncatewords: 18 }}</div>
<span class="proj-card-date"><i class="far fa-calendar-alt"></i> {{ post.date | date: "%B %Y" }}</span>
</div>
</div>
{% endfor %}
</div>