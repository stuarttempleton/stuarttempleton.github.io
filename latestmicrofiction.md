---
layout: raw
title: Latest Microfiction
permalink: /latestmicrofiction/
---

{% for post in site.categories["microfiction"] limit:1 %}
{{ post.title | upcase }}

{{ post.content | strip_html }}
{% endfor %}

-- Stuart Templeton (https://www.stuarttempleton.com)

