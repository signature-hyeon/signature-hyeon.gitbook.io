---
description: Resilience4j CircuitBreaker에 대해
---

# Resilience4j CircuitBreaker

## **1. 왜 맨날 나만 터지는 건데**

> 서킷 브레이커가 필요한 이유

**이럴 때 있잖아요**

* 외부 API가 응답 안 하는데
* 우리 서비스는 계속 때림
* 결국 우리 서비스도 터짐

**🧯 그래서 Circuit Breaker가 필요하다**

서킷 브레이커는 실패가 일정 기준을 넘으면 해당 요청을 일시적으로 끊는다.

바로 재시도하지 않고, 잠깐 기다렸다가 상태가 괜찮아지면 다시 연결을 시도한다.

즉,

* 무의미한 호출을 줄이고
* 전체 장애로 번지는 걸 막는다

**언제 쓰면 좋을까?**

* 외부 API가 불안정할 때
* 느린 응답 때문에 전체 서비스가 영향을 받을 때
* 일단 멈추고 상태를 본 다음 다시 시도하고 싶을 때

**정리하자면**

Circuit Breaker는 지속적인 실패를 빠르게 차단하고, 전체 시스템을 지키는 안전장치다.

## **2. 세상엔 3가지 상태가 있다**

> Closed, Open, Half-Open

서킷 브레이커는 단순히 차단하고 말자가 아니다. 상태를 보면서 똑똑하게 판단한다. 크게 보면 3가지 상태가 있다.

**Closed – 정상 상태**

* 모든 요청을 그대로 보낸다
* 실패율을 내부적으로 계속 기록한다
* 실패율이 임계치를 넘으면 Open 상태로 전환된다

**Open – 요청 차단**

* 모든 요청을 즉시 거절한다
* 외부 시스템에 부담을 주지 않기 위해 차단하는 상태
* 일정 시간 동안 이 상태를 유지한다

**Half-Open – 테스트 상태**

* 제한된 수의 요청만 보낸다
* 성공률이 높으면 Closed로 전환
* 실패가 반복되면 다시 Open 상태로 돌아간다

## **3. 세팅 잘못하면 진짜 터짐**

> CircuitBreakerConfig 뜯어보기

서킷 브레이커는 설정값에 따라 민감하게 반응하기도 하고, 너무 무뎌지기도 한다. 핵심 설정 몇 가지만 알아두면 된다.

#### **주요 설정값**

**failureRateThreshold**

* 실패율 임계값(%)
* 이 값을 넘으면 Closed → Open 상태로 전환
* ex) 50이면, 최근 요청 중 절반이 실패하면 차단 시작

**slidingWindowType**

* 실패율 계산 방식
* COUNT\_BASED: 고정된 요청 개수 기준 (ex. 최근 100건)
* TIME\_BASED: 일정 시간 기준 (ex. 최근 10초)&#x20;

**slidingWindowSize**

* 실패율을 계산할 때 참고할 요청 수나 시간
* 위 slidingWindowType에 따라 의미가 달라짐

**minimumNumberOfCalls**

* 실패율 계산을 시작하기 위한 최소 호출 수
* 너무 적은 샘플로 판단하지 않기 위한 값

**waitDurationInOpenState**

* Open 상태에서 Half-Open으로 전환되기까지 대기 시간
* 너무 짧으면 자꾸 호출하고, 너무 길면 복구가 늦어진다

**permittedNumberOfCallsInHalfOpenState**

* Half-Open 상태에서 허용할 테스트 요청 수
* 성공하면 Closed, 실패하면 다시 Open

#### 요약하면

| 설정값                                     | 설명                    |
| --------------------------------------- | --------------------- |
| `failureRateThreshold`                  | 실패율 기준 (%)            |
| `slidingWindowType`, `Size`             | 실패율 계산 방식과 범위         |
| `minimumNumberOfCalls`                  | 판단을 시작하기 위한 최소 호출 수   |
| `waitDurationInOpenState`               | Open 상태 유지 시간         |
| `permittedNumberOfCallsInHalfOpenState` | Half-Open에서 테스트할 호출 수 |

아래는 샘플로 작성해본 예시다.

```java
@Bean
public CircuitBreakerConfigCustomizer rssFetcherCircuit() {
    return CircuitBreakerConfigCustomizer.of("rssFetcher", builder -> builder
            .slidingWindowType(SlidingWindowType.COUNT_BASED)
            .failureRateThreshold(50)
            .slowCallRateThreshold(100)
            .slowCallDurationThreshold(Duration.ofSeconds(2))
            .minimumNumberOfCalls(5)
            .slidingWindowSize(10)
            .waitDurationInOpenState(Duration.ofSeconds(5))
            .permittedNumberOfCallsInHalfOpenState(2)
    );
}
```

| 설정 항목                                   | 값             | 설명                              |
| --------------------------------------- | ------------- | ------------------------------- |
| `failureRateThreshold`                  | `50%`         | 실패율이 50% 이상이면 **Open 상태 전환**    |
| `slidingWindowType`                     | `COUNT_BASED` | 최근 호출 **개수 기준**으로 실패율 계산        |
| `slidingWindowSize`                     | `10`          | 최근 **10번 호출**을 기준으로 상태 판단       |
| `minimumNumberOfCalls`                  | `5`           | 최소 5번 호출되기 전까지는 평가하지 않음         |
| `waitDurationInOpenState`               | `5초`          | Open 상태에서 **5초 후 Half-Open 전환** |
| `permittedNumberOfCallsInHalfOpenState` | `2`           | Half-Open 상태에서 **2번 테스트 호출 허용** |
| `slowCallDurationThreshold`             | `2초`          | 2초 넘으면 느린 호출로 간주                |
| `slowCallRateThreshold`                 | `100%`        | 느린 호출 비율이 100%면 실패로 처리          |

서킷 브레이커는 잘 써야 도움이 된다. 너무 민감하게 세팅하면 오히려 정상 서비스도 막는다.

## **4. 한 줄이면 끝남**

> 사용 방법 완전 정복

서킷 브레이커는 코드에서 직접 쓰거나, 애노테이션으로 간단하게 붙일 수 있다.

**1. @CircuitBreaker 애노테이션 (Spring Boot)**

```java
@CircuitBreaker(name = "rssFetcher", fallbackMethod = "fallback")
public String callExternalApi() {
    return restTemplate.getForObject("<https://external.api>", String.class);
}

public String fallback(Exception e) {
    return "fallback response";
}
```

* name은 설정에 등록한 이름
* fallbackMethod는 실패했을 때 대체할 메서드

**2. 함수 데코레이션 방식 (Java + Resilience4j Core)**

```java
CircuitBreaker cb = CircuitBreaker.ofDefaults("myApi");

Supplier<String> decorated = CircuitBreaker
    .decorateSupplier(cb, () -> externalService.call());

Try<String> result = Try.ofSupplier(decorated)
    .recover(throwable -> "fallback");
```

* 함수형 스타일로 wrapping
* 실패 처리 흐름을 recover()로 지정 가능

복잡해 보이지만, 결국 “실패하면 끊고, 대체할 방법을 미리 준비해두자” 가 핵심이다.

## **5. 망했을 때 플랜 B가 있어야지**

> Retry, TimeLimiter, Fallback

서킷 브레이커는 실패를 감지하고 차단하는 데 특화돼 있다. 하지만 그것만으로는 부족할 수 있다. Retry, TimeLimiter, Fallback 같은 도구들과 함께 써야 진짜 유용해진다.

**1. Retry**

* 일시적인 오류는 다시 시도해서 극복
* 예: 타임아웃, 네트워크 불안정

**2. TimeLimiter**

* 너무 오래 걸리는 요청은 끊어버림
* 예: 응답 없는 외부 API에 대한 방어

**3. Fallback**

* 실패했을 때 대체 로직을 실행
* 예: 캐시 데이터로 응답, 기본 메시지 반환 등

**실행 순서 요약**

1. Retry: 몇 번 더 시도해보고
2. TimeLimiter: 일정 시간 넘기면 끊고
3. CircuitBreaker: 실패가 쌓이면 차단하고
4. Fallback: 그래도 안 되면 기본값 반환

실패를 단순히 막는 게 아니라, 어떻게 대응할지를 미리 설계해두는 것이 중요하다.

## **6. 죽었는지 살았는지 어떻게 알아**

> 모니터링과 메트릭 확인법

서킷 브레이커는 상태 기반으로 동작하기 때문에 지금 어떤 상태인지, 얼마나 실패했는지 모르면 제대로 운영하기 어렵다. 그래서 모니터링이 꼭 필요하다.

**상태 확인 방법**

**1. Actuator 엔드포인트**

Spring Boot를 사용하면 /actuator/circuitbreakers 에서 현재 상태와 통계 정보를 바로 확인할 수 있다.

* 상태 (Closed / Open / Half-Open)
* 호출 수, 실패율, 마지막 상태 변경 시각 등

```yaml
management:
  endpoints:
    web:
      exposure:
        include: circuitbreakers
```

* [http://localhost:8080/actuator/health](http://localhost:8080/actuator/health)
* [http://localhost:8080/actuator/metrics](http://localhost:8080/actuator/metrics)
* [http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.state](http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.state)
* [http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.state?tag=name:rssFetcher\&tag=state=open](http://localhost:8080/actuator/metrics/resilience4j.circuitbreaker.state?tag=name:rssFetcher\&tag=state=open)

**2. Prometheus + Grafana**

* Resilience4j는 메트릭을 Micrometer로 노출
* Prometheus로 수집하고, Grafana로 시각화
* 서킷 브레이커별 호출 수, 실패율, 전이 횟수 등을 대시보드로 볼 수 있다

**3. CircuitBreakerRegistry**

Java 코드에서 직접 상태나 통계를 확인하고 싶을 때

```java
CircuitBreaker cb = registry.circuitBreaker("myApi");
cb.getMetrics().getFailureRate(); // 실패율
cb.getState(); // 현재 상태
```

**주의할 점**

* Open 상태가 지나치게 자주 발생하면 설정을 다시 봐야 한다
* Half-Open 이후 다시 Open으로 돌아간다면 → 외부 시스템이 아직 회복되지 않았다는 의미
* 장애 대응 과정에서 Fallback으로 대체된 경우도 로그로 추적해야 한다

모니터링이 없다면 서킷 브레이커는 그냥 호출이 안 되는 이유를 모르는 블랙박스가 된다.
