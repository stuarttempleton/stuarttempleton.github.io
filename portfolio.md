---
layout: page
title: Portfolio
permalink: /portfolio/
---

This is a collection of the work that exhibits something of value. While some of these projects are in varying states of completion and release, they are all strong examples of my professional design and development skills.

{::options parse_block_html="true" /}
<div class="highlights"> 
    {%- if site.categories.download.size > 0 -%} 
    {%- for post in site.categories.download -%}
    [![alt text]({{ post.image | escape}} "{{ post.title | escape}}"){:class="highlight"}]({{ post.url | relative_url }})
    {%- endfor -%}
    {%- endif -%}