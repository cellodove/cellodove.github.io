---
title: "컴퓨터 사이언스"
permalink: /categories/Cs/ 
layout: category
author_profile: true
sidebar_main: true
taxonomy: 컴퓨터 사이언스
---

{% assign posts = site.categories.Cs %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}