---
layout: page
title: Projects
icon: fas fa-robot
order: 2
---

<p>Here is a log of the autonomous systems, control frameworks, and robotics hardware I have developed.</p>

<div id="post-list" class="flex-grow-1 mt-4">
  {% for post in site.categories.Projects %}
    <article class="card-wrapper card mb-4">
      <a href="{{ post.url | relative_url }}" class="post-preview row g-0 flex-md-row" style="text-decoration: none;">
        
        {% if post.image %}
          <div class="col-12 col-md-5">
            <img src="{{ post.image.path | relative_url }}" alt="Preview" class="w-100 h-100" style="object-fit: cover; border-top-left-radius: 3px; border-bottom-left-radius: 3px; min-height: 200px;">
          </div>
        {% endif %}

        <div class="col-12 {% if post.image %}col-md-7{% endif %}">
          <div class="card-body d-flex flex-column h-100">
            <h2 class="card-title my-2 mt-md-0" style="color: var(--heading-color);">{{ post.title }}</h2>
            <div class="card-text content mt-0 mb-3 text-muted">
              <p>{{ post.content | strip_html | truncatewords: 25 }}</p>
            </div>
            <div class="post-meta flex-grow-1 d-flex align-items-end text-muted font-monospace" style="font-size: 0.85rem;">
              <i class="far fa-calendar fa-fw me-1"></i>
              <time>{{ post.date | date: "%B %d, %Y" }}</time>
            </div>
          </div>
        </div>
        
      </a>
    </article>
  {% endfor %}
</div>