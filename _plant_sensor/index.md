---
layout: collection
title: Plant Sensor Posts
categories: []
permalink: /:collection/index.html
---
{% assign collection = site.collections | where:"label", page.collection | first %}
{{ collection.description }}

# Posts
{% assign sorted = collection.docs | where:"layout", "post" | sort: 'date' %}
{% for item in sorted %}
* [{{ item.title }}]({{ item.url }})
> {{ item.excerpt }}
{% endfor %}