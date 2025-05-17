---
layout: about
title: About
---

{% assign seconds_since_graduate = site.time | date: '%s' | minus: 1718812800 %}
{% assign shit_days = seconds_since_graduate | divided_by: 86400 %}
{% assign shit_years = shit_days | divided_by: 365.0 | round %}

## 关于我

一无名小卒，本科毕业差不多{{ shit_years }}年，呼号**BD7PLV**。闲赋在家时喜欢折腾赛博垃圾，遂建次博客，记录点滴心得。

## 关于本站

内容随机，无定向，以技术为主，用大白话记录所学所思，以便日后查阅。

为保证阅读质量，本站所有图片均无水印，可随意用于非商业场合，转载请注明出处。

---

## About Me

A nobody, approximately {{ shit_years }} year(s) out of undergrad pretty much, likes to tinker with cyber trash when idle at home, so I built this blog to record bits and pieces of my experience.

## About This Site

Content is random, undirected, mainly technical, using plain language to record what I have learned and thought, for future reference.

To ensure the quality of reading, all images on this site are watermark-free and can be used freely for non-commercial purposes. Please indicate the source when reprinting.

---

## Contact

- Email: [{{ site.author.email }}](mailto:{{ site.author.email }})