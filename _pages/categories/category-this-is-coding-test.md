---
title: "이것이 코딩테스트다"
layout: archive
permalink: categories/this-is-coding-test
author_profile: true
sidebar_main: true
---

<!-- 공백이 포함되어 있는 카테고리 이름의 경우 site.categories['a b c'] 이런식으로! -->

***

📚 **<u>이것이 코딩 테스트다</u>** 나동빈 님의 '이것이 코딩 테스트다'책을 보고 정리한 내용입니다.<br><br>
<img src="https://user-images.githubusercontent.com/98803599/218809230-4f73bc18-c0ab-458d-b298-ad1d805db1d5.jpeg" height="20%" width="20%">
{: .notice--primary}

{% assign posts = site.categories['This is coding test'] %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}