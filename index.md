---
layout: default
title: Inicio
---

<section class="hero">
  <h1>Bienvenido a Picaro.dev</h1>
  <p>Últimos artículos:</p>
  <ul>
    {% for post in site.posts %}
      <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> – {{ post.date | date: "%-d %B, %Y" }}</li>
    {% endfor %}
  </ul>
</section>