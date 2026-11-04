---
layout: "single"
title: "Tags"
permalink: "/en/tags/"
lang: "en"
locale: "en-US"
translation_url: "/tags/"
classes: "wide"
description: "Sungwon Kim — backend engineering, measured performance, realtime systems, and recovery verification."
---


{% assign documents = site.projects | concat: site.reports %}
{% capture tag_names %}{% for document in documents %}{% for tag in document.tags %}{{ tag }}|{% endfor %}{% endfor %}{% endcapture %}
{% assign sorted_tags = tag_names | split: '|' | uniq | sort %}
{% for tag in sorted_tags %}{% unless tag == '' %}
<section class="taxonomy__section" id="{{ tag | slugify }}">
  <h2>{{ tag }}</h2>
  <ul>
  {% for document in documents %}{% if document.tags contains tag %}
    {% for translation in site.pages %}{% if translation.lang == 'en' and translation.translation_url == document.url %}
    <li><a href="{{ translation.url | relative_url }}">{{ translation.title }}</a></li>
    {% endif %}{% endfor %}
  {% endif %}{% endfor %}
  </ul>
</section>
{% endunless %}{% endfor %}
