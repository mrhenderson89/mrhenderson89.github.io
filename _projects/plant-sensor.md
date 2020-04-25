---
project-id: plant-sensor
layout: project
title: Plant Sensor
featured: true
featured-priority: 2
listing-priority: 2
---

{% assign collection = site.collections | where:"label", page.collection | first %}
{{ collection.description }}

# Posts
{% assign sorted = collection.docs | where:"layout", "post" | sort: 'date' %}
{% for item in sorted %}
* [{{ item.title }}]({{ item.url }})
> {{ item.excerpt }}
{% endfor %}