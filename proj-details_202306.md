# Project Details

- [Project Details](#project-details)
  - [발화생성](#발화생성)
  - [CoreLM](#corelm)
  - [BigModel](#bigmodel)
  - [Emotion Analysis](#emotion-analysis)



## 발화생성

- 프로젝트 목표 : 커스텀 분류기를 만들기 위해 사용자들이 직접 training set을 생성해야하는데, 이를 도와주는 tool로서 intent를 제공하면, training set을 대량 생산할 수 있는 tool 제작
- 프로젝트 내용
  - intent가 주어지면 training set을 생성하는 모델 제작
  - inference 최적화
    - seq2seq모델은 onnx porting을 위해, encoder-first decoder-decoder 구조로 세번 포팅이 필요하다.
    - 하지만 이러면 cpu thread를 너무 많이 차지하고, torch 대비 속도가 잘 나지 않는다.
    - 이를 [encoder-first decoder]-decoder 구조로 포팅하여, thread 수를 2/3배로 줄이고 결과적으로 속도향상을 할 수 있었다.
- keywords
  - bart
  - onnx

## CoreLM

- 프로젝트 목표 : 최신 pretrained language modelling의 연구 분야를 따라가고, 사내 한국어 pretrained language model들을 보유
- 프로젝트 내용
  - larger corpus, larger batch

    - plm의 트랜드 중 하나인 larger corpus를 제작하기 위해, 외부 데이터와 색인 데이터를 수집하고 전처리 함
    - horovod와 deepspeed를 이용한 multi-node, multi-gpu를 이용해 larger batch size로 모델을 학습할 수 있게 세팅 함
  - 수렴 안정성
    - dynamic batch size나 curriculum learning을 통해 수렴 안정성을 확보하여 large size 모델들도 큰 트러블슈팅 없이 좋은 성능을 낼 수 있었다.
  - tokenizer experiments & setting

    - 일반적으로 학습된 토크나이저, 형태소 분석된 코퍼스에서 학습된 토크나이저들, 형태소 분석 + subword signal를 추가한 코퍼스에서 학습된 토크나이저들을 비교 분석
    - 토크나이저 레벨의 차이는 성능상 큰 차이를 주지 않는다는 결론과, 그런 만큼 fine-tuning level에서 parsing하기 좋은 토크나이저가 좋은 토크나이저라는 결론을 얻음
  - sequence 2 sequence model - bart
    - data noising에 token deletion 추가하여 mrc 능력이 향상된 모델 제작
    - plm task을 multi-task( mlm, clm, text infilling )로 변경하여 전반적인 모델 성능 향상
- keywords
  - multi-node, multi-gpu training
  - horovod, deepspeed
  - dynamic batch size, curriculum learning
  - roberta, electra, bart, normformer

## BigModel

- 프로젝트 목표 : 사내 big model 기술 스탯 및 모델의 보유
- 프로젝트 내용
  - megatron 레포를 이용한 bigmodel 학습
  - lazy loader 개발
    - huggingface datasets 패키지에서 제공하는 데이터를 읽는 방식은 초기 로딩 시간이 느림
    - pyarror 패키지를 이용하여 로딩 타임이 필요하지 않게 lazy loader를 구현
- keywords
  - megatron
  - zero optimization
  - huggingface datasets pacakge

## Emotion Analysis

- 프로젝트 목표 : 주어진 문장에 어울리는 감정을 추출하고, 어울리는 스티커를 추천해주는 인공지능 api를 제작
- 프로젝트 내용
  - multi-label classification
    - 기존에는 문장에 대응하는 감정 클래스가 1개만 있을 수 있다고 생각하여 uni-label로 문제를 접근
    - 하나의 문장에서 발견될 수 있는 감정 클래스들은 여러가지가 있을 수 있음에도 불구하고, 모델의 출력이 인위적으로 정하게 된 label과 맞지 않아서 모델이 틀렸다고 평가함
    - 실제로는 문장에 대응하는 감정을 여러가지가 있을 수 있다고 판단하여, 데이터셋을 uni-label이 아닌 multi-label로 설정하고, metric을 accuracy가 아닌 precision at K와 recall at K로 수정함
    - 모델을 학습시키기 위한 loss를 target metric에 맞춰, loss를 설계함
  - ranking loss

    - multi-label classification 문제를 '주어진 텍스트에 대해서 적절한 감정을 순서대로 나열하는 것'이라는 관점에서 봄
    - '순서대로'라는 것을 생각해봤을 땐 해당 문제를 ranking을 어떻게 주는가 라고 생각해볼 수 있다.
    - learning to rank 분야의 기술들을 적용해봄
    - 성능이 높아지거나 낮아지지 않았음
- keywords
  - multi-label classification
  - ranking loss

