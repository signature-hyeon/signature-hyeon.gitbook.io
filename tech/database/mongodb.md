---
description: MongoDB에 대해 알아보자
---

# MongoDB

## MongoDB 란?

> “빠르고 유연한 NoSQL”을 지향

비관계형 데이터베이스로, 문서 지향(Document-Oriented) 데이터베이스이다. 이러한 구조는 **자주 변경되는 데이터 스키마**나 **대용량 쓰기 작업**이 필요한 환경에 적합하다. 관계형 데이터베이스에서 MySQL이 대표적인 것처럼, MongoDB는 NoSQL 데이터베이스 중 가장 널리 사용되는 대표적인 시스템이다.

## 특징

MongoDB는 내부적으로 **바이너리 형태(BSON)**&#xB85C; 직렬화해서 저장한다. 각 필드의 이름, 값, 타입을 함께 저장해서 스키마를 몰라도 데이터를 읽을 수 있다.

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-06 235623 (2).png" alt=""><figcaption></figcaption></figure>

#### BSON

Binary JSON의 약자로, JSON 데이터의 바이너리 표현이다. JSON보다 빠르게 파싱되고, 더 적은 공간을 사용할 수 있다. 또한, ObjectId, Date, Decimal128 등 [더 많은 데이터 유형](https://www.mongodb.com/docs/manual/reference/bson-types/#std-label-bson-types)을 지원하여 보다 정규한 값으로 저장하고, 효율적인 연산을 위해 최적의 사용법도 있다.

**MongoDB는 왜 BSON으로 저장할까?**

BSON의 인코딩 방식은 유형과 길이 정보도 인코딩하여 JSON에 비해 훨씬 더 빠르게 탐색하기 때문이다.&#x20;

```json
{"hello": "world"} →
\x16\x00\x00\x00           // total document size
\x02                       // 0x02 = type String
hello\x00                  // field name
\x06\x00\x00\x00world\x00  // field value
\x00                       // 0x00 = type EOO ('end of object')
 
{"BSON": ["awesome", 5.05, 1986]} →
\x31\x00\x00\x00
 \x04BSON\x00
 \x26\x00\x00\x00
 \x02\x30\x00\x08\x00\x00\x00awesome\x00
 \x01\x31\x00\x33\x33\x33\x33\x33\x33\x14\x40
 \x10\x32\x00\xc2\x07\x00\x00
 \x00
 \x00
```

예를 들어, JSON에서는 "부터 "까지 문자열 전체를 읽어야 "world"라는 걸 알 수 있고, BSON은 "\x06"를 읽자마자 "아, 뒤에 6비트짜리 문자열이 오겠구나!"하고 바로 6바이트만 읽고 넘어갈 수 있다.

<table><thead><tr><th width="163">포맷</th><th>방식</th></tr></thead><tbody><tr><td>JSON</td><td>책을 읽다가 “끝 문장 부호가 언제 나올지” 몰라서 한 글자씩 계속 읽음</td></tr><tr><td>BSON</td><td>처음에 “이 문장은 6글자야” 라고 써 있어서, 6글자만 읽고 바로 다음으로 넘어감</td></tr></tbody></table>

## 데이터 구조

관계형 DB와 달리 테이블 대신 컬렉션, 행(row) 대신 도큐먼트(document)를 사용한다.

| 개념            | RDBMS            | MongoDB                 |
| ------------- | ---------------- | ----------------------- |
| **데이터 묶음 단위** | Table            | Collection              |
| **데이터 단위**    | Row (정형화된 레코드)   | Document (JSON/BSON 형식) |
| **데이터 구성 요소** | Column (고정된 스키마) | Field (key-value 형태)    |

## 장단점

장점

* 스키마리스 모델링
* 고가용성(수평적 확장)에 유리함

단점

* 스키마가 없어서 데이터 일관성 보장 어려움
* 조인 지양, 트랜잭션 덜 엄격함

### 수평적 확장이 유리한 이유?

**수평 확장을 전제로 설계됨**

샤딩 기능을 DB 자체에서 기본 지원하기 때문에 클러스터에 샤드를 더 붙이기만 하면 된다. MongoDB는 한 개 문서(document)에 필요한 데이터가 다 들어있기 때문에, 데이터를 나눠서 저장해도 JOIN이 필요 없다. 즉, 샤드 간 통신 줄이고, 독립적 처리가 가능하다.

**RDBMS에서는 어떻게 확장할까?**

RDBMS는 기본적으로 수직 확장을 전제로 설계되어 있기 때문에 확장이 필요할 경우 보통 CPU나 메모리(RAM)를 늘리는 방식으로 처리한다. 만약 수평 확장(horizontal scaling)이 필요하다면, 샤딩을 직접 구현하고 데이터를 분산 처리하는 로직을 애플리케이션 또는 프록시 레이어에서 구성해야 한다.



## 참고

* [https://www.mongodb.com/resources/basics/json-and-bson](https://www.mongodb.com/resources/basics/json-and-bson)
*
