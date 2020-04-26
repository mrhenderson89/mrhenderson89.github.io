---
title: Test
subtitle: A series of room-sensor units I created using Arduino boards
layout: project
icon: fa-pencil-alt
order: 3
---

{%- if page.content != "" -%}
		{%- include section.html title=page.title photo=page.cover-photo photo-alt=page.cover-photo-alt auto-header=page.auto-header content=page.content -%}
	{%- endif -%}
	
	<!-- Posts List -->
	{% for post in site.categories[page.title] %}
		{%- capture _title -%}
			<a href="{{- post.url | absolute_url -}}">{{- post.title -}}</a>
		{%- endcapture -%}
		{%- capture _subtitle -%}
			{% if post.author -%}{{- post.author }} | {% endif %}
			{{- post.date | date_to_long_string -}}
		{%- endcapture -%}
		{%- capture _excerpt -%}<p>{{- post.excerpt | strip_html | truncatewords: 100 -}}</p>{%- endcapture -%}
		{%- capture _link -%}<a href="{{- post.url | absolute_url -}}">read more</a>{%- endcapture -%}
		{%- assign _content = _excerpt | append: _link -%}
		{%- include section.html title=_title subtitle=_subtitle content=_content -%}
	{% endfor %}