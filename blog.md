---
layout: page
title: 	Blog
permalink: /blog/
---

<ul class="post-list">
{% for post in site.posts %}
	<li class="post-li">
		<span class="post-meta">{{ post.date | date_to_string}}</span>
  		<h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
  		<span class="post-excerpt">{{ post.excerpt }}</span>
	</li>
{% endfor %}
</ul>