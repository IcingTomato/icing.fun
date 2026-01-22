---
layout: default
title: About
---

{% assign seconds_since_graduate = site.time | date: '%s' | minus: 1718812800 %}
{% assign shit_days = seconds_since_graduate | divided_by: 86400 %}
{% assign shit_years = shit_days | divided_by: 365.0 | round %}

<h2>关于我</h2>
<p>一无名小卒，本科毕业差不多{{ shit_years }}年，呼号<strong>BD7PLV</strong>。闲赋在家时喜欢折腾赛博垃圾，遂建次博客，记录点滴心得。</p>

<h2>关于本站</h2>
<p>内容随机，无定向，以技术为主，用大白话记录所学所思，以便日后查阅。</p>
<p>为保证阅读质量，本站所有图片均无水印，可随意用于非商业场合，转载请注明出处。</p>
<hr>

<h2>About Me</h2>
<p>A nobody, approximately {{ shit_years }} year(s) out of undergrad pretty much, likes to tinker with cyber trash when idle at home, so I built this blog to record bits and pieces of my experience.</p>

<h2>About This Site</h2>
<p>Content is random, undirected, mainly technical, using plain language to record what I have learned and thought, for future reference.</p>
<p>To ensure the quality of reading, all images on this site are watermark-free and can be used freely for non-commercial purposes. Please indicate the source when reprinting.</p>
<hr>

<h2>Contact</h2>
<ul>
    <li>Email: <a class="btn-classic mb-2 white bg-dark-blue" href="mailto:{{ site.author.email }}">{{ site.author.email }}</a></li>
</ul>
