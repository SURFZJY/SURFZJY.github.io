---
layout: page
permalink: /blog/
title: Blog
description: Notes on multimodal and omni-modal language models.
nav: true
nav_order: 1
pagination:
  enabled: true
  collection: posts
  permalink: /page/:num/
  per_page: 5
  sort_field: date
  sort_reverse: true
  trail:
    before: 1
    after: 3
---

{% if site.display_tags.size > 0 or site.display_categories.size > 0 %}
<p class="post-tags">
  {% for tag in site.display_tags %}
    <a href="{{ tag | slugify | prepend: '/blog/tag/' | relative_url }}"><i class="fa-solid fa-hashtag fa-sm"></i> {{ tag }}</a>
    {% unless forloop.last %}&nbsp;{% endunless %}
  {% endfor %}
  {% if site.display_categories.size > 0 and site.display_tags.size > 0 %}&nbsp;&middot;&nbsp;{% endif %}
  {% for category in site.display_categories %}
    <a href="{{ category | slugify | prepend: '/blog/category/' | relative_url }}"><i class="fa-solid fa-tag fa-sm"></i> {{ category }}</a>
    {% unless forloop.last %}&nbsp;{% endunless %}
  {% endfor %}
</p>
{% endif %}

<ul class="post-list">
  {% if page.pagination.enabled %}
    {% assign postlist = paginator.posts %}
  {% else %}
    {% assign postlist = site.posts %}
  {% endif %}

{% for post in postlist %}
    {% assign read_time = post.content | number_of_words | divided_by: 180 | plus: 1 %}
    {% assign year = post.date | date: "%Y" %}
    {% assign tags = post.tags | join: "" %}
    {% assign categories = post.categories | join: "" %}
    <li>
      <h3><a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p>{{ post.description }}</p>
      <p class="post-meta">{{ read_time }} min read &nbsp;&middot;&nbsp; {{ post.date | date: "%B %d, %Y" }}</p>
      <p class="post-tags">
        <a href="{{ year | prepend: '/blog/' | relative_url }}"><i class="fa-solid fa-calendar fa-sm"></i> {{ year }}</a>
        {% if tags != "" %}
          &nbsp;&middot;&nbsp;
          {% for tag in post.tags %}
            <a href="{{ tag | slugify | prepend: '/blog/tag/' | relative_url }}"><i class="fa-solid fa-hashtag fa-sm"></i> {{ tag }}</a>
            {% unless forloop.last %}&nbsp;{% endunless %}
          {% endfor %}
        {% endif %}
        {% if categories != "" %}
          &nbsp;&middot;&nbsp;
          {% for category in post.categories %}
            <a href="{{ category | slugify | prepend: '/blog/category/' | relative_url }}"><i class="fa-solid fa-tag fa-sm"></i> {{ category }}</a>
            {% unless forloop.last %}&nbsp;{% endunless %}
          {% endfor %}
        {% endif %}
      </p>
    </li>
{% endfor %}
</ul>

{% if page.pagination.enabled %}
{% include pagination.liquid %}
{% endif %}
