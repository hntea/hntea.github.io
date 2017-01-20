---
layout: page
title: Category 
excerpt: "An archive of Category sorted by date."
search_omit: true
image:
  feature: so-simple-sample-image-11.jpg
---

{% for category in site.categories %}
<h2>{{ category | first }}</h2>
<ul class="Category-list">
    {% for post in category.last %}
        <li>{{ post.date | date:"%d/%m/%Y"}}<a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
{% endfor %}
