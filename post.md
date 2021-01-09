---
title: Post
layout: page
show_sidebar: false
hero_height: is-small
hero_darken: true
---

<ul id="archive">
{% for post in site.posts %}
  {% capture y %}{{post.date | date:"%Y"}}{% endcapture %}
  {% if year != y %}
    {% assign year = y %}
    <h3 class="blogyear">{{ y}}</h3>
  {% endif %}
<li class="archiveposturl"><span><a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a></span><br/>
<span class = "postlower">

{{ post.date | date: '%d %b %Y' }}
</span>

</li>
{% endfor %}
</ul>

