# 아이템48 스트림 병렬화는 주의해서 적용하라

주류 언어 중 동시성 프로그래밍 측면에서 자바는 항상 앞서갔다.
- 1996년 : 스레드, 동기화, wait/notify 지원
- Java 5 : 동시성 컬렉션인 java.util.concurrent 라이브러리와 실행자(Executor) 프레임워크 지원
- Java 7 : 고성능 병렬 분해 프레임워크인 포크-조인(fork-join) 패키지 추가
- Java 8 : parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림 지원


## 동시성 프로그래밍을 할 때는 
- `안전성(safety)`과 `*응답 가능(liveness)` 상태를 유지해야 한다.
  - 안전 실패(safety failure) :  오동작 
- 병렬 스트림 파이프라인 프로그래밍에서도 마찬가지
- 자바는 parallel() 메서드 호출로 스트림 파이프라인을 사용할 수 있다.


## [예시 코드] 메르센 소수 구하기
```java
    public static void main(String[] args) {
        StreamParallel streamParallel = new StreamParallel();

        long start = System.currentTimeMillis();
        streamParallel.primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
        long end = System.currentTimeMillis();
        System.out.println("running time : " + (end - start));
    }

    public Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
```

### 파이프라인 병렬화가 불가능 한 경우 성능 개선이 되지 않는다.
아래 두 가지 경우에는 파이프라인 병렬화로 성능 향상을 기대하기 어렵다.
1. 데이터 소스가 Stream.iterate인 경우
2. 중간 연산으로 limit()를 사용하는 경우
  - 파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.
  - 그러나 메르센소수를 찾을 때마다 그 전 소수를 찾을 때보다 두배 정도 더 오래 걸린다. 원소 하나를 계산하는 비용이 대략 그 이전까지의 원소 전부를 계산한 비용을 합친 것만큼 든다는 뜻이다.

---

## 병렬화를 사용하기 위한 조건

### 1. **스트림 소스 조건**
스트림의 소스가 아래 자료형일 때 병렬화의 효과가 가장 좋다. 
- ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스
- 배열, int 범위, long 범위일 때 

#### 해당 자료구조 특징
1. **데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서** 일을 다수의 스레드에 분배하기에 좋다.  
  - 나누는 작업은 Spliterator가 담당
  - Spliterator 객체는 `Stream` 또는 `Iterable`의 **spliterator()** 메서드로 얻어올 수 있다.
2. **참조 지역성(locality of referece)**이 뛰어나다 
  - 메모리에 연속해서 저장되어 있다.
  - 참조 지역성이 낮으면 `스레드`는 데이터가 주 메모리에서 **캐시 메모리로 전송되어 오기를 기다리며** 대부분 시간을 멍하니 보내게 된다.
  - `참조 지역성`은 `벌크 연산`을 병렬화할 때 아주 `중요한 요소`로 작용한다
  - 기본 타입 배열에서는 참조가 아닌 데이터 자체가 메모리에 연속해서 저장되기 때문에 참조 지역성이 가장 뛰어난 자료구조이다.

### 2. **스트림 파이프라인의 종단 연산 동작 방식**
종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하므로 스트림 파이프라인의 종단연산의 동작방식이 병렬 수행 효율에 영향을 준다. 

- 종단 연산 중 병렬화에 가장 적합한 것은 **축소(reduction)**
  - reduction은 **파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업**이다.
  - **Stream.reduce(), min(), max(), sum(), count()** 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해 수행한다.
- **anyMatch, allMatch, noneMatch**처럼 `조건에 맞으면 바로 반환되는 메서드`도 병렬화에 적합하다.
- 반면, `가변 축소(Mutable Reduction)`을 수행하는 Stream의 collect 메서드는 병렬화에 `적합하지 않다.` 컬렉션들을 합치는 부담이 크기 때문이다.


## 효과있는 스트림 파이프라인 병렬화 코드
```java
//before 
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
                     .mapToObj(BigInteger::valueOf)
                     .filter(i -> i.isProbablePrime(50))
                     .count();
}


//after : parallel()으로 스트림 파이프라인 사용
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
                     .parallel()
                     .mapToObj(BigInteger::valueOf)
                     .filter(i -> i.isProbablePrime(50))
                     .count();
}
```

## Random한 수의 경우
- 무작위 수들로 이뤄진 스트림을 병렬화할 경우 ThreadLocalRandom(혹은 Random) 보다는 `SplittableRandom 인스턴스를 이용`하자. 
- `SplittableRandom`  : `병렬화를 위해 설계`된 것이라 병렬화 하면 성능이 선형으로 증가한다. 
- `ThreadLocalRandom` : `단일 스레드에서 사용`하고자 만들어 졌다.
- `Random` : 모든 연산을 `동기화`하기 때문에 병렬 처리하면 최악의 성능을 보일 것이다.

---

## 스트림 병렬화는 **성능 최적화 수단**
- 위의 조건을 만족하더라도 병별화에 드는 추가 ㅈ비용을 상쇄하지 못하면 성능향상은 미미할 수 있다.
- **변경 전후로 반드시 성능테스트를 진행**하여 병렬화를 사용할 가치가 있는지 확인해야 한다. **운영 시스템과 흡사한 환경에서 테스트**하는 것이 좋다.
- [ ] 보통은 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행되므로 (같은 스레드 풀을 사용) 잘못된 파이프라인 하나가 다른 부분의 성능에까지 악영향을 줄 수 있음을 유념.
- 조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 볼 수 있다. 머신러닝과 데이터 처리 같은 특정 분야에서 성능 개선 기대.


