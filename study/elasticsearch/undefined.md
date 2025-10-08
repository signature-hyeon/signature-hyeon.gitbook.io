---
description: Elasticsearch는 어떻게 빠르게 검색할 수 있을까?
---

# 색인과 검색 원리

## 색인이란?

Elasticsearch는 데이터를 저장할 때 **색인**한다고 표현한다. 이 때 역색인(inverted index) 구조에 저장한다. 역색인에 대해 알아보자.

### 역색인이란? (inverted index)

단어 기준으로 문서를 찾기 쉽게 미리 색인을 만들어 놓은 구조이다. '어떤 단어가 어떤 문서에 등장하는가'를 빠르게 찾기 위해 만들어졌다. 예를 들어, 단어 -> 문서 ID 목록 이런 식으로 미리 색인을 만들어둬서 특정 단어가 포함된 문서를 빠르게 찾을 수 있다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 어떻게 빠르게 검색할 수 있을까?

Elasticsearch는 역색인 구조로 저장하는 과정을 통해 빠르게 검색할 수 있다. 이때 내부적으로 텍스트 분석 과정을 거쳐 저장한다.

Elasticsearch에서 텍스트가 저장될 때 다음과 같은 과정을 거친다.

1. _Tokenizer: 토큰화_
2. _Normalization_
   1. _소문자 변환 (Quick -> quick)_
   2. _어근 추출 (foxes -> fox)_
   3. _동의어 처리 (jump, leap -> jump)_
3. _역색인 구조로 저장_

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

이처럼 Elasticsearch는 문장을 토큰화하고 불필요한 정보를 정제한 뒤, 역색인 구조로 저장함으로써 방대한 데이터 속에서도 원하는 정보를 빠르게 찾아낼 수 있다.



## 참고

* [https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-overview.html)
* [https://velog.io/@yeonns/how-full-text-search-works](https://velog.io/@yeonns/how-full-text-search-works)
* [https://develop-tracking.tistory.com/187](https://develop-tracking.tistory.com/187)
