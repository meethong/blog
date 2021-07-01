---
layout: default
permalink: /categories.html
---

<section class="archive-main">
{% assign sorted_categories = site.categories | sort %}
{% for category in sorted_categories %}
<h3>{{ category | first }}</h3>
<ol class="posts-list" id="{{ category[0] }}">
{% for post in category.last %}
<ul class="posts-list-item">
<span class="posts-list-meta">{{ post.date | date:"%Y-%m-%d" }}</span>
 <a class="posts-list-name" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
</ul >
{% endfor %}
</ol>
{% endfor %}
</section>



