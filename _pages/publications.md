---
layout: page
permalink: /publications/
title: Publications & Patents
nav: publications 
description: <nobr>*</nobr> denotes equal contribution or joint lead authorship.
nav: true
nav_order: 1
years: [2025, 2023, 2020]
---

{% include bib_search.liquid %}

<br/>
{% for y in page.years %}
  <div class="row m-0 p-0" style="border-top: 1px solid #ddd; flex-direction: row-reverse;">
    <div class="col-sm-1 mt-2 p-0 pr-1">
      <h3 class="bibliography-year">{{y}}</h3>
    </div>
    <div class="col-sm-11 p-0">
      {% bibliography -f papers -q @*[year={{y}}]* %}
    </div>
  </div>
{% endfor %}

