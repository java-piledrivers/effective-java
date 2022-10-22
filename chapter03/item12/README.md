# [아이템 12] toString을 항상 재정의하라


### toString은 간결하면서 사람이 읽히 쉬운 형태의 유익한 정보를 반환해야 한다.
이 메서드는 PhoneNumber@adbbd처럼 단순히 클래스_이름@16진수로_표시한_해시코드를 반환한다.   
toString 메서드의 재정의를 잘 해놓는다면 클래스를 좀 더 좋게 사용할 수 있다. 예를 들어 디버깅 또는 로깅 할 때 해당 객체의 정보를 쉽게 파악 할 수 있다.

### 포맷 문서화
포맷을 명시한다면 명확하고 사람이 읽을 수 있게 된다.    
그러나 포맷을 한번 명시하면 평생 그 포맷에 얽매이게 된다.   
이를 사용하는 프로그래머들이 그 포맷에 맞춰 파싱하고 새로운 객체를 만들고 영속 데이터로 저장하는 코드를 작성할 것이다. 만약 포맷을 바꾼다면 기존 사용하던 코드들은 엉망이 될 것이다.

#### 명시하는 경우
```java
/**
* 이 전화번호의 문자열 표현을 반환한다.
* 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
* XXX는 지역코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
* 각각의 대문자는 10진수 숫자 하나를 나타낸다.
* 
* 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
* 앞에서부터 0으로 채워나간다. 예컨데 가입자 번호가 123이라면
* 전화번호의 마지막 네 문자는 "0123"이 된다.
*/
@Override 
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

#### 명시하지 않는 경우
```java
/**
* 이 약물에 관한 대략적인 설명을 반환한다.
* 다음은 이 설명의 일반적인 형태이나,
* 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
* 
* "[약물 #9: 유형-사랑, 냄새=테러빈유, 겉모습=먹물]"
*/
@Override
public String toString() { ... }
```

### toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하는 것이 좋다.
toString에 포함되는 정보는 다른 API를 통해 가져갈 수 있도록 각 필드의 getter 메서드를 제공해야 한다.     
클래스의 정보를 필요로 하는 개발자들은 굳이 toString 정보를 파싱할 필요 없이 각 필드를 접근할 수 있는 API를 통해 정보를 얻을 수 있다.


### 결국 toString은 디버깅 용도로만 사용하는 것이 좋다.
PhoneNumber의 경우와 같이 문자열을 만들어서 사용하려면 toString보다는 별도의 메서드를 만들어 사용하는 것이 좋다. 
https://stackoverflow.com/questions/19911290/java-tostring-for-debugging-or-actual-logical-use   
https://stackoverflow.com/questions/563676/is-tostring-only-useful-for-debugging
