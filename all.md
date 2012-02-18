---
title: All posts
layout: default
---

# All posts

<ul>
{% for post in site.posts %}
<li>
<a href="{{ post.url }}">{{ post.title }}</a><br>
<span class="entry-meta">Published: {{ post.date | date: "%B %d, %Y" }}</span>
</li>
{% endfor %}
</ul>
