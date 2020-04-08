---
layout: page
title: Playable Downloads
permalink: /downloads/
list_tag: download
---

{%- if site.categories.download.size > 0 -%}
    {%- for post in site.categories.download -%}
    <table class="section">
        <tr>
            <td width="40%" valign="top">
            <a href="{{post.remote_url | escape}}"><img class="full" src="{{post.image | escape}}"/></a>
            </td>
            <td>
            <div class="content title">{{post.title | escape}}</div>
            <div class="content description">{{post.description | escape}}
            </div>
            <div class="content brief">
                <p><b>Contributors</b> {{post.contributors | escape}}</p>
                <p><b>Created At</b> {{post.created_at | escape}}</p>
                <p><b>Role</b> {{post.role | escape}}</p>
                <p><b>Platform</b> {{post.platform | escape}}</p>
            </div>
            </td>
        </tr>
    </table>
    {%- endfor -%}
{%- endif -%}
