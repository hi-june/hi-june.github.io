---
title: "Voice Pocket"
layout: archive
permalink: categories/Voice_Pocket
author_profile: true
sidebar_main: true
---

<!-- 공백이 포함되어 있는 카테고리 이름의 경우 site.categories['a b c'] 이런식으로! -->

***

😆 **<u>Voice Pocket</u>** 프로젝트에 관한 포스트 모음입니다!<br><br>
아래 이미지를 클릭하여 Voice Pocket의 깃허브 Repository를 구경해보세요!<br>
<a href="https://github.com/voicepocket" target="_blank">
	<img src="https://avatars.githubusercontent.com/u/121807294?s=400&u=324b4034dd575db5f0f0633d32167ee4cb1f4783&v=4" height="20%" width="20%">
<a>
{: .notice--primary}

{% assign posts = site.categories.Voice_Pocket %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}