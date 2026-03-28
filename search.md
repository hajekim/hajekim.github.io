---
layout: page
permalink: /search
---

#### Looking for something?

<div class="category-filter">
{% assign all_categories = site.posts | map: "categories" | flatten | uniq | sort %}
{% for category in all_categories %}
  <button class="category-tag" onclick="filterByCategory('{{ category }}')">{{ category }}</button>
{% endfor %}
</div>

{% include search.html %}
