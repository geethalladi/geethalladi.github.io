---
layout: page
title: Tags
permalink: /tags/
---

<ul>
{% assign sorted_tags = site.tags | sort %}

{% for tag in sorted_tags %}
<li>
<a href="#{{ tag[0] }}">{{ tag[0] }}</a> ({{ tag[1].size }})
</li>
{% endfor %}
</ul>
