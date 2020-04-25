---
layout: collection
title: Plant Sensor
categories: [Plant Sensor]
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