# Project Details

- [Project Details](#project-details)
  * [Reformer](#reformer)
  * [Micro-blog texts에 스티커 추천을 위한 딥러닝 기반의 API 제작](#micro-blog-texts에-스티커-추천을-위한-딥러닝-기반의-api-제작)
  * [EmotionGIF Challenge](#emotiongif-challenge)
  * [AI Challenge - Question & Answering task](#ai-challenge---question--answering-task)
  * [기업 분석을 위한 뉴스 비지도 클러스터링](#기업-분석을-위한-뉴스-비지도-클러스터링)


## Reformer

- 프로젝트 목표 : Transformer의 구조를 효율적으로 변환한 구조인 Reformer를 이해하고 pytorch 구현체를 tensorflow로 포팅하기
- 프로젝트 내용
    - Transformer based model의 메모리 낭비를 줄인 Reformer를 tensorflow로 구현
        - Backward 연산에 필요한 각 레이어의 output을 저장하지 않기 위해 reverisble net을 사용
        - Attention layer의 결과를 dense layer에 통과시킬 때, 몇 개의 chunk로 나눠서 계산
        - Locally-sensitive hashing attention을 통해 한 token에 대한 attention value를 몇 개의 token들의 attention value로 근사
- keywords
    - Reformer
    - PyTorch, TF1

## Micro-blog texts에 스티커 추천을 위한 API 제작

- 프로젝트 목표 : 유저가 작성한 메세지와 함께 보낼 수 있는 스티커를 추천하는 딥러닝 엔진 제작하기
- 프로젝트 내용
    - 주어진 텍스트와 함께 보낼 수 있는 스티커 카테고리를 예측하여 해당 카테고리의 스티커를 추천
    - 주어진 텍스트에 어울리는 스티커 카테고리가 여러가지일 수 있음을 고려하여, multi-label classification 태스크로 문제 전환하여 모델 제작
    - multi-label의 reduction 방법으로 OVA를 채택하고, metric으로는 p@k를 채택
    - 모델의 후보로서 Deepmoji, XLNet, DialoGPT, RoBERTa을 고려하고, inference 시간이 빠른 Deepmoji와 metric 스코어가 높았던 RoBERTa를 선택
    - 예측 결과는 [UI 상 적절하다고 판단되는 3개 카테고리들(top 3)]과 [class 별 precision 값이 0.6 이상이 되게 하는(top 1 기준) threshold 이상 카테고리들]의 교집합으로 출력
    - API화 하기 위해 tensorflow-serving과 flask를 이용
- keywords
    - Deepmoji, Roberta
    - TF2-Keras, Tensorflow-serving, Flask

## EmotionGIF Challenge

- Challenge : 트위터 post-reply pair 텍스트를 보고, reply에 사용된 GIF의 카테고리들을 예측하는 챌린지
- Challenge 내용
    - 모히톡 서비스에서 사용하고 있는 multi-label classification의 수준 및 성능을 객관적으로 측정할 수 있다고 생각되어 참가
    - 챌린지에서 사용하는 metric이 r@k라는 점을 이용하기 위해, autoencoding 모델인 RoBERTa, autoregressive 모델인 DialoGPT, permutation language modeling으로 학습한 XLNet의 조합들을 관찰 - (RoBERTa와 DialoGPT 조합의 성능이 높게 나옴)
    - multi-label의 reduction 방법으로 PAL-n을 채택
    - 데이터를 5-fold한 뒤, 4 fold로 학습하고 1 fold로 검증하여 early stop(한 모델 당 5개 버전의 모델을 학습)
- keywords
    - Dialogpt, XLNet, Roberta
    - TF2-Keras, TPU

## AI Challenge - Question & Answering task

- Challenge - 뉴스 기사와 질문 pair를 보고 답변을 기사 내에서 찾는 챌린지
- Challenge 내용
    - 해당 대회에서는 대회 데이터 외의 데이터로 학습된 pre-trained 모델들을 사용할 수 없다는 규정이 있어 기존 웹 상의 훈련된 transformer based model들을 fine-tuning할 수 없었음
    - 주어진 데이터로 ALBERT의 masked language modeling과 transformer 모델 이전에 사용되던 BiDAF와 DocQA의 구현을 병렬적으로 실행
    - ALBERT의 훈련은 주어진 데이터로 충분히 학습되지 않아 f1 score 결과가 낮게 나왔고, BiDAF와 DocQA 중에는 BiDAF가 근소하게 좋은 성적을 내어 BiDAF를 선택하여 제출
- keywords
    - ALBERT, BiDAF, DocQA
        - BiDAF - [word embeding + chan cnn] + [highway * 2] + bi-lstm + attention + bi-lstm * 2 + dense
        - DocQA - BiDAF(bi-gru) + self-attention + bi-gru + dense
    - PyTorch

## 기업 분석을 위한 뉴스 비지도 클러스터링

- 프로젝트 목표 : 특정 query에 대한 뉴스 기사들을 군집화 후 대표 뉴스를 추출하고 시간순으로 출력하여 사용자가 query에 대한 정보를 쉽고 빠르게 얻을 수 있는 시스템 제작하기
- 프로젝트 내용
    - 맡은 부분 : [키워드 검색 결과] + [카테고리 분류]가 된 뉴스 집합을 [비지도 클러스터링을 통해 비슷한 내용의 뉴스끼리 군집화]하고 [군집들 각각의 대표 뉴스를 추출]하는 파트
    - 각 뉴스들을 TF-IDF 벡터화를 통해 embedding space로 보내고, 군집 수와 군집의 밀도를 지정하지 않아도 되는 OPTICS 알고리즘을 통해 뉴스들을 군집화
    - 하나의 군집에서 top 10 단어를 이용하여 BM25 score를 통해 대표 뉴스를 추출
- keywords
    - OPTICS
    - sklearn