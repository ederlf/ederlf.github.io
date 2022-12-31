---
layout: page
permalink: /publications/
title: Publications
description: Publications from the time I built stuf for research in computer networks. I even got a PhD from it. 
years: [2020, 2019, 2018, 2017, 2016, 2014, 2013, 2011]
nav: true
nav_order: 1
---
<!-- _pages/publications.md -->
<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
