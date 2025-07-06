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
        {% for post in site.posts %}
        <li><a href="{{'/_posts/test-post.md'   | relative_url }}">{{ post.title }}</a> – {{ post.date | date: "%-d %B, %Y" }}</li>
        {% endfor %}
    </ul>
<a href="#features" class="btn">Working On</a>
</section>
