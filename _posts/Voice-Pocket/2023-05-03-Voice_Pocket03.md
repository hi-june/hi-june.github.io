---
title:  "[Voice Pocket] TTS 작업 속도 개선기"
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
## 음성 합성을 위한 과정
사용자의 요청에 따라 음성 파일을 합성하는 전반적인 과정은 다음과 같다.  

1) 클라이언트 요청  
2) 해당 음성 모델로부터 Systhesizer object initialize  
```python
synthesizer = Synthesizer(
            tts_checkpoint = f"{tts_model_file_path}",
            tts_config_path = f"{tts_config_path}"
        )
```

3) 2에서 만든 Synthesizer object로부터 tts 함수 호출(wav 파일 생성)  
```python
symbol = synthesizer.tts_config.characters.characters   # symbol set
text = normalize_text(text, symbol) # text normalize
wav = synthesizer.tts(text, None, None) # conver text to speech
synthesizer.save_wav(wav, wav_path) # write wav file
```

4) 생성된 wav 파일을 gcs(google cloud storage) bucket에 저장  
```python
def upload_wav_to_bucket(wav_path, email, uuid):
    storage_client = storage.Client()
    
    bucket_name = BUCKET_NAME
    source_file_name = wav_path
    destination_blob_name = f'{email}/{uuid}.wav'
    
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(destination_blob_name)

    blob.upload_from_filename(source_file_name)
```


## 문제는 너무 느리다는 것..
음성 하나 합성하는데 걸리는 시간이 너무 느리다는 것이 계속 걸렸다.  

2번과 3번 과정이 가장 핵심적인 부분이었고, 각각 과정(특히, 2번)들에서 많은 시간이 소요되었다.  

애초에 `coqui-tts`에서 제공하는 라이브러리를 사용하여 음성 합성을 진행하고 있기 때문에,  
위의 과정 중 3번에 해당하는 text를 기반으로 wav 파일을 합성하는 데 걸리는 시간은 건드리기 힘들었다.  

> 따라서, 작업 요청마다 Synthesizer instance가 initialize되는 시간만 날려버려도, 시간을 상당히 많이 줄일 수 있을 것이라 판단했고, 해당 부분에 cache를 적용하기로 했다.  


## redis cache
`redis`는 간단히 설명하자면 key-value 형식으로 데이터를 저장하는 in-memory 기반의 데이터베이스이다.  

애초에 gpu 서버에서 음성 학습을 시키고, 모델이 만들어질 때,  
해당 모델 파일로부터 synthesizer를 initialize하여 synthesizer object를 redis 캐시에 저장해두고,  
필요한 순간에 가져다가 쓸 수 있다면, 요청마다 initiaize되는 시간을 줄일 수 있을 것이라고 생각했다.  

정리하자면,  
- 음성 모델 학습이 끝나고  
1) 음성 모델 합성이 완료되면 해당 모델을 기반으로 synthesizer object를 initialize.  
2) synthesizer object를 redis에 저장  

- 음성 합성 요청 시  
1) 클라이언트 요청  
2) redis에 접근하여 미리 저장해둔 synthesizer objcet를 불러옴  
3) 불러온 object로 TTS 수행  


## class의 instance를 redis에 저장할 수 있을까?
`redis`에서는 value로 다양한 data type을 지원한다.  
대표적으로 string을 지원하므로 python의 pickle 모듈을 사용하여 객체를 문자열로 직렬화하여 저장하는 방식을 사용했다.  

```python
import pickle
import redis

# 레디스 연결
rd = redis.Redis(host='{host_name}', port=port_num, db=db_num)

# synthesizer initialize
synthesizer = Synthesizer(
            tts_checkpoint = f"{tts_model_file_path}",
            tts_config_path = f"{tts_config_path}"
        )

# synthesizer 객체를 pickle로 serialize하여 redis에 저장
serialized_syn = pickle.dumps(synthesizer)
rd.set(email, serialized_syn)
    
# redis에 저장해둔 객체를 불러옴
serialized_syn = rd.get(email)
syn = pickle.loads(serialized_syn)

symbol = syn.tts_config.characters.characters
text = normalize_text(text, symbol)
wav = syn.tts(text, None, None)
syn.save_wav(wav, WAV_PATH)
```


## 수행 결과
같은 길이의 문장을 기준으로 캐시를 적용하기 전과 후를 비교해보았다.  

- 적용 전  
![image](/assets/images/Voice_Pocket/3-1.png)<br>

- 적용 후  
![image](/assets/images/Voice_Pocket/3-2.png)<br>

cache를 적용하여 해당 작업을 21초에서 8초로 줄일 수 있었다!