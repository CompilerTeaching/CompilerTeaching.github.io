---
title: News
layout: default
published: false
---

{% for post in site.posts %}
##[{{ post.title }}]({{ site.baseurl}}{{ post.url }})
<p class="post-meta">{{ post.date | date: "%b %-d, %Y" }} • {% if post.author %}{{ post.author }}{% else %}{{ site.author }}{% endif %}{% if post.meta %} • {{ post.meta }}{% endif %}</p>
{{ post.excerpt }}
{% endfor %}

