---
description: Elasticsearch에 대해 알아보자
---

# Elasticsearch 란?

## Elasticsearch 란?

Apache Lucene 기반으로 한 무료 검색 및 분석 엔진이다. ELK 스택에서 저장, 분석을 담당하고 있다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### ELK 스택이란?

데이터를 집계, 분석, 시각화 기능을 제공하기 위한 플랫폼이다. 이 중 Elasticsearch는 전체 스택의 중심이며 가장 중요한 역할을 하고 있다.

* **Elasticsearch**: 검색, 분석을 담당하는 엔진
* **Logstash**: 데이터를 수집, 가공하여 Elasticsearch로 넘기는 파이프라인
* **Kibana**: Elasticsearch에 저장된 데이터를 시각화해주는 대시보드 도구

### Apache Lucene 이란?

Jave 기반 텍스트 검색 엔진 라이브러리로, 인덱싱 및 검색, 토큰화 등 대규모 텍스트 데이터를 빠르게 검색할 수 있도록 만들어진 도구이다.

### 활용 분야

* 검색 엔진
* 로그, 메트릭 수집 및 추적
* 실시간 데이터 집계 및 대시보드

## Elasticsearch 특징

### 빠른 검색 성능 (full-text search)

Lucene은 역색인 구조로 데이터를 저장하여 가공된 텍스트를 검색한다. 이를 **full-text search**라고 한다. 데이터를 역색인 구조로 저장해 실시간으로 검색과 분석도 가능하게 한다.

### Restful API

데이터의 CRUD 작업을 http Restful API 를 통해 수행한다. 모든 시스템에서 기본적으로 사용하는 프로토콜이므로 누구나 이해하고 쓸 수 있는 구조로 사용할 수 있다.

### 오픈소스

Elasticsearch의 핵심 기능들은 Apache 2.0 라이센스로 배포되고 있다. Elastic Stack의 모든 프로덕트는 깃헙 레포지토리에서 찾을 수 있고, Releases를 통해 새로운 버전과 변경된 내용이 정리된 릴리즈 노트도 확인할 수 있다.

## 어떻게 빠르게 검색할 수 있을까?

일반적인 DB는 데이터를 저장한 순서대로 읽지만, Elasticsearch는 검색을 빠르게 하기 위해 역색인 구조를 사용한다. 그렇다면 역색인 구조는 무엇일까?

### 역색인 (inverted index) 이란?

단어 기준으로 문서를 찾기 쉽게 미리 색인을 만들어 놓은 구조이다. 예를 들어, 단어 -> 문서 ID 목록 이런 식으로 미리 색인을 만들어둬서 특정 단어가 포함된 문서를 빠르게 찾을 수 있다.

### RDB로는 위 기능을 구현하지 못할까?

RDB는 like 또는 full-text 인덱스를 사용해서 구현할 수 있지만, 텍스트가 많아질수록 느리고 분석기가 없어 형태소 분석이 필요한 언어(한글 등)에는 적절하지 않다.

Elasticsearch는 Analyzer, Tokenizer, Filter를 통해 분할하여 저장 및 유사도 점수를 부여하기 때문에 동의어나 유의어도 지원한다. 문자열 기반의 검색 최적화를 위해 별도로 설계된 인덱스 구조이므로 더 빠르고 커스텀화한 검색이 가능하다.

## 한계점

* 데이터 변경에 효율적이지 않다. (문서 전체 교체 방식, deleted 표시 후 주기적으로 삭제)
* 쓰기보다 읽기에 최적화되어 있다. (배치로 데이터를 수집 후 검색하는 용도가 적합)
* 조인과 트랜잭션이 없다.

## Tip

* 잦은 데이터 변경에 취약하므로 변경이 거의 없는 데이터일 때 사용을 고려하자.
* 데이터 정규화보다 비정규화(nested or 중복)해서 넣는게 일반적이다.
* mapping은 한번 만들면 거의 수정이 어렵가 때문에 초기 설계가 중요하다.

## 참고

* [https://aws.amazon.com/ko/what-is/elk-stack/](https://aws.amazon.com/ko/what-is/elk-stack/)
* [https://esbook.kimjmin.net/](https://esbook.kimjmin.net/)
* [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/analysis-overview.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/analysis-overview.html)
