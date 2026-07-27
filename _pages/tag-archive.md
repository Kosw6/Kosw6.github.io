---
title: "Tags"
layout: single
permalink: /tags/
author_profile: false
---

{% assign tag_documents = site.posts | concat: site.projects | concat: site.reports %}
{% capture tag_names -%}
  {%- for document in tag_documents -%}
    {%- for tag in document.tags -%}{{ tag }}|{%- endfor -%}
  {%- endfor -%}
{%- endcapture %}
{% assign sorted_tags = tag_names | split: "|" | uniq | sort %}

<ul class="taxonomy__index">
  {% for tag in sorted_tags %}
    {% if tag != "" %}
      {% assign tag_count = 0 %}
      {% for document in tag_documents %}
        {% if document.tags contains tag %}
          {% assign tag_count = tag_count | plus: 1 %}
        {% endif %}
      {% endfor %}
      <li>
        <a href="#{{ tag | slugify }}">
          <strong>{{ tag }}</strong>
          <span class="taxonomy__count">{{ tag_count }}</span>
        </a>
      </li>
    {% endif %}
  {% endfor %}
</ul>

{% for tag in sorted_tags %}
{% if tag != "" %}
<section id="{{ tag | slugify }}" class="taxonomy__section">
  <h2 class="archive__subtitle">{{ tag }}</h2>
  <div class="entries-list">
{% for document in tag_documents %}
{% if document.tags contains tag %}
{% assign post = document %}
{% include archive-single.html %}
{% endif %}
{% endfor %}
  </div>
  <a href="#page-title" class="back-to-top">맨 위로 이동</a>
</section>
{% endif %}
{% endfor %}
