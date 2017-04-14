---
layout: page
title: Guestbook | 留言板
description: Welcome to leave a message
tags: [Message]
imagefeature: wj/27.jpg
comments: true
custom_css: walk
---

<div id="walk-container">
  <div id="walk"></div>
</div>


> 走过，路过，言语两句：

{% if site.disqus_shortname and page.comments %}
  <div id="disqus_thread"></div>
  {% include disqus_comments.html %}
{% endif %}
