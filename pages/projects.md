---
layout: page
title: "A list of projects"
permalink: "/projects/"
order: 100
---

<ul>
  {% for project in site.projects | sort: 'listing-priority' %}
    <li>
      <a href="{{ project.url }}">{{ project.title }}</a>
      - {{ project.title }}
    </li>
  {% endfor %}
</ul>