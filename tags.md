---
layout: page
title: Tags
permalink: /tags/
---

<ul>
{% raw %}{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
  <li>
    {{ tag[0] }} ({{ tag[1].size }})
  </li>
{% endfor %}{% endraw %}
</ul>
