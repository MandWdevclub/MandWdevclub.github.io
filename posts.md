---
layout: page
title: Posts
date: 2015-09-29
updated: 2015-09-29
permalink: /posts/
---

<div class="container">

<h1>Posts</h1>

<a href="/2015-09-29-adding-a-post">Want to add or edit a post?</a>

<ul class="collection">
{% for post in site.posts %}
    <li class="collection-item"><a href="{{ post.url }}"><div>
      <span class="title">{{ post.title }}</span><br>
      <span class="grey-text text-darken-1">{{ post.date | date: '%B %d, %Y' }}</span><br>
      {% for category in post.categories %}
        <div class="chip">{{ category }}</div>
      {% endfor %}
      </p>
    </div></a></li>
{% endfor %}
</ul>

</div>
