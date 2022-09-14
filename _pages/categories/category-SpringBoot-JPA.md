---
title: "Spring Boot & JPA"
layout: archive
permalink: categories/SpringBoot-JPA
author_profile: true
sidebar_main: true
---

<!-- 공백이 포함되어 있는 카테고리 이름의 경우 site.categories['a b c'] 이런식으로! -->

***

📚 **<u>Inflearn</u>** 김영한 님의 '실전! 스프링 부트와 JPA 활용 - 웹 애플리케이션 개발'강의를 듣고 정리한 내용입니다.<br><br>
강의 정보가 궁금하신 분은 아래 이미지를 클릭하세요!<br>
<a href="https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1" target="_blank">
	<img src="https://cdn.inflearn.com/public/courses/324119/course_cover/07c45106-3cfa-4dd6-93ed-a6449591831c/%E1%84%80%E1%85%B3%E1%84%85%E1%85%AE%E1%86%B8%205%20%E1%84%87%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A1%204.png" height="20%" width="20%">
<a>
{: .notice--primary}

{% assign posts = site.categories.SpringBoot-JPA %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}