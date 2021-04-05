---
layout: default
permalink: /archive.html
---



<section class="archive-main">
{% assign count = 1 %}
{% for post in site.posts reversed %}
    {% assign year = post.date | date: '%Y' %}
    {% assign nyear = post.next.date | date: '%Y' %}
    {% if year != nyear %}
        {% assign count = count | append: ', ' %}
        {% assign counts = counts | append: count %}
        {% assign count = 1 %}
    {% else %}
        {% assign count = count | plus: 1 %}
    {% endif %}
{% endfor %}


{% assign counts = counts | split: ', ' | reverse %}
{% assign i = 0 %}

{% assign thisyear = 1 %}

{% for post in site.posts %}
    {% assign year = post.date | date: '%Y' %}
    {% assign nyear = post.next.date | date: '%Y' %}
    {% if year != nyear %}
        {% if thisyear != 1 %}
            </ol>
        {% endif %}

<h3>{{ post.date | date: '%Y' }} ({{ counts[i] }})</h3>

        {% if thisyear != 0 %}
            {% assign thisyear = 0 %}
        {% endif %}
        <ol class="posts-list">
        {% assign i = i | plus: 1 %}
    {% endif %}

<ul class="posts-list-item">
<span class="posts-list-meta">{{ post.date | date:"%m-%d" }}</span>
<a class="posts-list-name" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
</ul >
{% endfor %}
</ol>
</section>