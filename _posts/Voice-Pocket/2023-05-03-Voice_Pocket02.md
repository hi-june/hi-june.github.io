---
title:  "[Voice Pocket] spring boot에서 celery 활용하기"
excerpt: "Voice Pocket의 프로젝트"

categories:
  - Voice_Pocket
tag:
  - Info

toc: true
toc_sticky: true

date: 2023-05-03
last_modified_at: 2023-05-03
---
## 굳이 django를 써야할까..?
한이음 멘토님과 프로젝트 관련 이야기를 하다가, 이런 이야기가 나왔다.

<u>나</u>: 음성 합성 관련 라이브러리가 python으로 쓰여서 백엔드 서버도 `django`를 사용하고 있습니다 ㅠㅠ  
<u>멘토님</u>: 음.. 서버가 요청을 `rabbitMQ` 같은 message queue에 publish하면, python으로 받아서 처리하면 되니 굳이 서버까지 python으로 만들 필요는 없지요  
<u>나</u>: (생각해보니 그렇네…!)


## 서버 개발 언어에 굳이 제약을 걸 필요가 없겠구나!
아무래도 이제는 `django`보단 `spring boot`가 더 익숙하기도 하고, 요즘 적극적으로 공부하고 있는 프레임워크이기 때문에 굳이 python을 사용해야하는 상황이 아니라면 `django`를 고집하고 있을 이유도 없겠다는 생각에 기존에 구현해뒀던 서버를 `spring boot`로 변경하게 되었다.

> spring boot(java, TTS 요청) → RabbitMQ → Celery(python, TTS 수행)  

![image](/assets/images/Voice_Pocket/2-3.png)<br>

## python celery를 사용하기
python에서는 기본적으로 동기식으로 일을 처리한다.  
![image](/assets/images/Voice_Pocket/2-2.png)<br>

즉, 5명의 사용자가 TTS를 요청한다면, 5번 째 요청한 사람은 앞에 4명의 작업이 끝날 때까지 기다리고 있어야한다.  
해당 작업은 시간이 꽤 걸리는 작업이기 때문에 작업을 비동기적으로 처리해줄 방법이 필요했고, 그래서 찾은 것이 `celery`이다.


```python
from celery import Celery

# celery app 정의
app = Celery()
app.config_from_object("celery_config")

# task 정의
@app.task
def add(x, y):
    result = x + y
    return result

# task 비동기 호출
task = add.delay(1,2)
```
celery는 task queue를 사용하고, 이런 식으로 python으로 정의해놓은 함수를 이용해 쉽게 task를 정의 및 호출이 가능하다는 것이 장점이다.  


## spring boot에서 celery로 작업 넘기기
![image](/assets/images/Voice_Pocket/2-1.png){: width="60%" height="60%"}<br>
단순할 줄 알았는데, 생각보다 많은 삽질을 했다.  

처음에는 `spring boot`에서 해당 queue에 json 형식으로 인자를 주고, 그 인자를 받아 pasing하는 task를 정의하려고 했으나, 생각해보니 task를 정의한다고 해도 그걸 어디서 호출할 것인지도 모호했고,  

그냥 celery queue에 message를 실어주면, 알아서 worker가 수행해주지 않을까 싶었는데,  
queue에 실리는 task에는 일정 형식이 필요했고, 그게 맞지 않으면, worker가 그냥 message queue에서 해당 message를 삭제해버렸다.  

관련해서 이것저것 찾아보다가 `medium`에서 해답을 얻을 수 있었다.  


## celery bootsteps 사용하기
> You may want to embed custom Kombu consumers to manually process your messages.  
요약하자면 '사용자가 custom한 message consumer를 추가할 수 있다.  

```python
import redis, json

from celery import Celery
from celery import bootsteps
from kombu import Consumer, Exchange, Queue

rd = redis.Redis(host='host_name', port=port_num, db=db_num)

queue = Queue("queue_name", Exchange("exchange_name"), "key_name")

app = Celery()
app.config_from_object("celery_config")

# TTS 정의
@app.task
def text_to_speech(uuid, email, text):
    from tts_process import add_synth, is_set, make_tts
    if not is_set(email):
        add_synth(email)

    make_tts(email, uuid, text)
        
    return f"{email}/{uuid}.wav"


# Decalring the general input message handler
class InputMessageHandler(object):
    def handle(self, body):
        _type = body["type"]
        
        # if you want to accept only specific type of message, you need below
        if _type == "ETL":
            ETLMessageHandler().handle(body)
        
        # if body is json type, you need below
        '''
        body_json = json.loads(body)  
        _type = body_json["type"]

        if _type == "ETL":
            ETLMessageHandler().handle(body_json)
        '''


# Declaring the ETL message handler
class ETLMessageHandler(object):
    def handle(self, body):
        print(f"Working on ETL for message: {body}")
        # TODO: Calling out your Celery tasks here
        _uuid = body["uuid"]
        _email = body["email"]
        _text = body["text"]
        
        # 정의한 task를 호출
        task = text_to_speech.delay(_uuid, _email, _text)
        _task_json = json.dumps({"task_id":task.id})
        rd.set(_uuid, _task_json)


# Declaring the bootstep for our purposes
class InputMessageConsumerStep(bootsteps.ConsumerStep):
    def get_consumers(self, channel):
        return [Consumer(channel,
                         queues=[queue],
                         callbacks=[self.handle_message],
                         accept=["json"])]

    def handle_message(self, body, message):
        InputMessageHandler().handle(body)
        message.ack()


app.steps["consumer"].add(InputMessageConsumerStep)
```

rabbitMQ 브로커에 kombu로 만든 queue를 얹어놓고, 거기에 spring boot에서 message를 publish하는 전략이다.  
body에서 필요한 인자들을 뽑아내, handle 메소드 안 쪽에서 미리 정의해놓은 celery task를 호출하는 방식으로 이루어진다.  

개인적으로 느끼기에 많이 특이한 방식이기도 하고, ai 관련 라이브러리들은 보통 python으로 많이 짜여있어서,  
서버는 서버 따로, ai 관련 작업은 해당 작업 따로 관리하고자 할 때 유용하게 사용할 수 있을 것 같아 따로 정리해두었다.  

혹시 비슷한 작업을 하고자한다면 참고하는 것도 좋을 것 같다 :)  
<https://github.com/hi-june/boot_rabbit_celery> [Spring boot + RabbitMQ + Celery demo project]

## Reference
1) <https://medium.com/python4you/calling-celery-tasks-not-from-python-699bd635d317> [Calling Celery Tasks not from Python]  