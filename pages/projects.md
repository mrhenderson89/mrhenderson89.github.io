---
layout: page
title: "A list of projects"
permalink: "/projects/"
---

<ul>
  {% for project in site.projects %}
    <li>
      <a href="{{ project.url }}">{{ project.title }}</a>
      - {{ project.title }}
    </li>
  {% endfor %}
</ul>