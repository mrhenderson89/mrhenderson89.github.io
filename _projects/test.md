---
project-id: test
layout: project
title: Test
featured: true
featured-priority: 3
listing-priority: 3
---

{% assign collection = site.collections | where:"label", page.collection | first %}
{{ collection.description }}

# Posts
{% assign sorted = collection.docs | where:"layout", "post" | sort: 'date' %}
{% for item in sorted %}
* [{{ item.title }}]({{ item.url }})
> {{ item.excerpt }}
{% endfor %}