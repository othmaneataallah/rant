---
layout: default
title: rant
---

## How to read

- Click on a post to read it.  
- Newest posts show up first.  
- Explore whatever interests you.  

## Posts

Posts count: {{ site.posts | size }}

<ul>
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <small>— {{ post.date | date: "%Y-%m-%d" }}</small>
  </li>
{% endfor %}
</ul>

---

*That’s all.*