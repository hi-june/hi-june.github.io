---
title:  "[Voice Pocket] Message Queue 뒷 단으로 보낸 작업의 결과를 확인하는 법"
excerpt: "Voice Pocket의 프로젝트"

categories:
  - Voice_Pocket
tag:
  - Info

toc: true
toc_sticky: true

date: 2023-05-19
last_modified_at: 2023-05-19
---
## Message Queue 뒷 단에서 수행되는 task에 대해서 서버에서 어떻게 확인할 수 있을까..?
음성 합성을 위한 작업은 `python` 컨테이너에서 수행되므로 `spring boot` 서버에서 컨테이너 쪽으로 작업 메시지를 넘기면 그 작업 결과에 대해 클라이언트가 알 수가 없었다.  

하지만 wav 파일 생성을 마치면 해당 wav파일을 `gcs` 버킷에 업로드하기 때문에, 클라이언트에서 `gcs`로부터 다운로드 받기 위해서는 해당 작업이 끝난 시점을 알아야만 했다.  

따라서, 며칠 동안 머리를 싸맨 결과 나름의 해결책을 고안했다.  


## 끝났는지 계속 물어보기(Polling 방식)  
![image](/assets/images/Voice_Pocket/4-1.png)<br>

당최 해결법이 떠오르지 않다가, 문득 하나의 생각이 머리를 스쳤다.  

> celery backend가 있구나?  

celery는 작업의 결과를 backend에 저장할 수 있다.  
이 특성을 이용해서 result backend를 spring boot에서 조회하면서  
task id에 해당하는 작업 결과가  
1. result backend에 없으면 작업이 끝나지 않은 것이니 다시 조회 요청  
2. result backend에 있으면 작업이 끝난 것이니 url을 통해 음성 파일 다운로드 수행  

이런 방식으로 클라이언트에서 작업의 결과를 받아볼 수 있겠다 싶었고, 실제로 구현까지 완료했다.  
다만, 뇌피셜(?)로 나온 구현 방법이라 확신도 서지 않았고, 구현하면서도 바로 문제점이 보였다.  


## Polling 방식에서 나온 여러 문제들
여기서, `polling` 방식이란,  
> 일정한 주기(특정한 시간)을 가지고 서버와 응답을 주고 받는 방식이다.  

음성 합성 작업의 진행상황을 파악하기 위해 작업이 끝날 때까지 일정 간격으로 서버에 request를 요청하는 방식이니 `polling` 방식이라고 볼 수 있다.  
그리고 이 방법으로는 클라이언트와 서버 둘 다 문제가 있었다.  

- 클라이언트(Flutter)
    - TTS 작업을 요청한 뒤, 완료 response를 얻어낼 때까지, foreground로 request를 지속해서 보내야하기 때문에 다른 작업을 못했다.  

- 서버(Spring Boot)
    - 폴링의 주기가 짧으면 서버의 성능에 부담이 간다. (오버헤드/트래픽)
    - 주기가 길면 실시간성이 떨어진다.

> 음성 합성에는 대략 8~9초 정도가 소요되는데, 그 시간동안 앱에서 로딩바를 보고있어라..?  

생각만으로도 부정적 경험이다.  
안타깝게도, 별다른 방법이 떠오르지 않아, 저번처럼 멘토님께 SOS를 요청했다.  


## backend server를 producer면서 consumer로도 활용할 수 있지요
![image](/assets/images/Voice_Pocket/4-2.png)<br>

말 그대로다..  
너무 단방향으로만 사고를 했던 것 같다.  
작업이 끝났으면 끝났다는 message를 역방향으로 Queue에 실어주면 그만이었다.  
`spring boot`와 `celery worker`를 각각 producer와 consumer 역할을 모두 수행하도록 구성할 수도 있다는 생각은 못 했던 것 같다.  

작업이 완료되면 작업이 완료되었다는 알림을 `FCM`(Firebase Cloud Messaging)을 통해 client application에 알려주기로 했다.  

## 번외) notify는 어디서 해줘야되지..?  
사실 처음에는 `celery worker`가 wav 파일 생성을 마치고 `FCM`에 push request를 보내는 일까지 시키려고 했다.  
하지만, '각 작업에 대한 책임은 분리하는게 좋다'고 조언해주셔서 `FCM`에 관한 작업은 `spring boot` 서버에서 수행하도록 구성했다.  

> 이로써 python 컨테이너는 오로지 음성 합성에 대한 작업만,  
spring boot 서버는 클라이언트와 소통하는 작업만 수행할 수 있도록 구현되었다.  

클래스, 메소드는 기능단위로 딱딱 분리해가며 구현해왔으면서 순간 별 생각없이 구현할 뻔 했다..ㅎㅎ