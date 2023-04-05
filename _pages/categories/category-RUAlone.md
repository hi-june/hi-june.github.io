---
title: "RUAlone(혼자왔니)"
layout: archive
permalink: categories/RUAlone
author_profile: true
sidebar_main: true
---

<!-- 공백이 포함되어 있는 카테고리 이름의 경우 site.categories['a b c'] 이런식으로! -->

***

😆 **<u>RUAlone(혼자왔니)</u>** 프로젝트에 관한 포스트 모음입니다!<br><br>
아래 이미지를 클릭하여 RUAlone 깃허브 Repository를 구경해보세요!<br>
<a href="https://github.com/TecheerBootios" target="_blank">
	<img src="https://avatars.githubusercontent.com/u/115468028?s=400&u=3cb3b00ef2efacc945d6ad1bc1116d89f798a2e7&v=4" height="20%" width="20%">
<a>
{: .notice--primary}

{% assign posts = site.categories.RUAlone %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}