---
layout: default
title: Blog
permalink: /blog/
---

# Blog

Older posts were moved here from the Blogger site that used to run on www.budgester.com, and keep their original addresses.

{% for post in site.posts %}- {{ post.date | date: "%-d %B %Y" }}: [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}

[How this site is written]({{ '/how-this-site-is-written/' | relative_url }}): how these posts come out of real work, with an AI assistant.
