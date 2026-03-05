---
layout: default
title: Inicio
---
<section class="hero">
    <h1 class="glitch" data-text="WELCOME , let's LEARN">WELCOME , let's write about<span class="changing-words">
    <span>DEVOPS</span>
    <span>AI</span>
    <span>DEV</span>
    <span>INFRA</span>
    <span>MATH</span>
    </h1>
    <p>Últimos artículos:</p>
    <ul>
        {% for post in site.posts limit:3 %}
        <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> – {{ post.date | date: "%-d %B, %Y" }}</li>
        {% endfor %}
    </ul>
<a href="{{ '/blog' | relative_url }}" class="btn">Blog</a>
</section>
