# 아이템 68. 일반적으로 통용되는 명명 규칙을 따르라 

자바 플랫폼의 명명 규칙은 잘 정립되어 있으며, 그중 많은 것이 자바 언어 명세에 기술되어 있다. 자바의 명명 규칙은 크게 `철자`와 `문법`, 두 범주로 나뉜다.

## 1. 철자 규칙

특별한 이유가 없는 한 반드시 따라야 한다.

### 패키지와 모듈

- 점(.)으로 구분하여 계층적으로 짓는다.
- 모두 소문자 알파벳 혹은 숫자로 구성한다.
- 조직 바깥에서도 사용될 패키지라면, 인터넷 도메인 이름을 역순으로 사용한다.
- 점(.)으로 구분되는 각 요소는 보통 8자 이하 짧은 단어/약어로 한다.
- 예외 적으로 표준 라이브러리와 선택적 패키지들은 각각 java, javax로 시작한다.

### 클래스와 인터페이스

- 하나 이상의 단어로 구성하며, 각 단어는 대문자로 시작한다.
- max, min처럼 널리 통용되는 줄임말을 제외하고는 단어를 줄여 쓰지 않도록 한다.
- 약어의 경우 각 약자의 시작과 끝을 명확하게 알 수 있도록 첫 글자만 대문자로 하는 경우가 많다 (HttpUrl, HTTPURL 어떤쪽이 나아보이는가?)

### 메서드와 필드

- 클래스 명명 규칙과 첫글자를 소문자로 쓴다는 점만 다르다.
- 첫 단어가 약자라면 단어 전체가 소문자여야한다.
- 상수 필드는 구성하는 단어 모두를 대문자를 사용하며, 단어 사이는 밑줄(_)로 구분한다.

### 타입 매개변수

보통 한 문자로 표현

- T : 임의의 타입
- E : 컬렉션 원소의 타입
- K : 맵의 키
- V : 맵의 값
- X : 예외
- R : 메서드의 반환 타입

## 2. 문법 규칙

철자 규칙과 비교하면 더 유연하고 논란도 많다.

| 구분 | 규칙 | 예 |
| --- | --- | --- |
| 객체를 생성할 수 있는 클래스 | 단수 명사 / 명사구 | Thread, PriorityQueue, ChessPiece |
| 객체를 생성할 수 없는 클래스 | 복수형 명사 | Collectors, Collections |
| 인터페이스 | 클래스와 동일하게 / ~able, ~ible | Collection, Comparator, Runnable, Iterable |
| 애너테이션 | 자유롭게 | BindingAnnotation, Inject, Singleton |
| 동작을 수행하는 메서드 | 동사 / (목적어를 포함한) 동사구 | append, drawImage |
| boolean 값을 반환하는 메서드 | is~, has~ | isDigit, isEmpty, hasText |
| 반환 타입이 boolean이 아닌 메서드 | 명사 / 명사구 / get~ | size, hashCode, getTime |
| 객체의 타입을 바꿔서 다른 객체를 반환하는 메서드 | to(Type) | toString, toArray |
| 객체의 내용을 다른 뷰로 보여주는 메서드 | as(Type) | asList |
| 객체의 값을 기본 타입 값으로 반환하는 메서드 | (type)Value | intValue |
| 정적 팩터리 메서드 |  | from, of, valueOf, instance, getInstance |


## 3. 정리

표준 명명 규칙을 체화하여 자연스럽게 베어 나오도록 하자.

철자 규칙은 직관적이라 모호한 부분이 적은 데 반해, 문법 규칙은 더 복잡하고 느슨하다.

자바 언어 명세의 말을 인용하자면 "오랫동안 따라온 규칙과 충돌한다면 그 규칙을 맹종해서는 안된다." 상식이 이끄는 대로 따르자.
