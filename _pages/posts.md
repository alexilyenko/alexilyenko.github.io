---
title:  
layout: archive
permalink: /
author_profile: true
comments: false
---

<ul>
  <h1>Recent Posts</h1>
  {% for post in site.posts %}
    {% unless post.next %}
      <font color="#778899"><h2 class="archive__subtitle">{{ post.date | date: '%Y %b' }}</h2></font>
    {% else %}
      {% capture year %}{{ post.date | date: '%Y %b' }}{% endcapture %}
      {% capture nyear %}{{ post.next.date | date: '%Y %b' }}{% endcapture %}
      {% if year != nyear %}
        <font color="#778899"><h2 class="archive__subtitle">{{ post.date | date: '%Y %b' }}</h2></font>
      {% endif %}
    {% endunless %}
   {% include archive-single.html %}
  {% endfor %}
</ul>
