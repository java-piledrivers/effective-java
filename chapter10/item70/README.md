# 아이템 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

![png;base64669bf411b9dbea2c](https://user-images.githubusercontent.com/30370933/212199813-0defbe22-f8be-4581-a93d-1f06603258d0.png)

### Exception은 검사(Checked) 예외, Error와RuntimeException은 비검사(unChecked) 예외

#### 검사(Checked) 예외

- 개발자가 명시해야하는 Exception, 어플리케이션 수행 중에 일어날법한 예외를 검사하고 대비하라는 목적으로 사용
- 과도하게 예외 검출을 사용하면 시스템의 성능이 떨어짐 [아이템71]

#### 비검사(unChecked) 예외

- 시스템적인 예외를 의미하며, 개발자가 예외를 잡지 말아야하는 심각한 상황에서 발생하는 예외
- 프로그래밍 오류를 나타낼 떄는 RuntimeException를 사용
ex) 배열의 인덱스가 0에서 '배열 크기-1'를 벗어났을때 ArrayIndesOutofBoundsException이 발생함

JVM의 자원 부족, 불변식 깨짐 등 더이상 실행될 수 없는 상황에서 Error를 사용


-Error 클래스를 상속해 하위 클래스를 만드는일은 자제해야함
-> 구현해야하는 비검사 예외는 모두 RuntimeException을 상속한 하위 클래스여야 한다.

### Exception, RuntimeException, Error를 상속하지 않는 throwable을 만들 수도 있지만 사용하지 않아야한다.
-> throwable를 사용할때랑 비교해서 나은게 없으므로 혼란만 가중시킨다.

### 검사 예외를 구현할때는 추가적으로 복구에 필요한 정보를 알려주는 메서드도 함께 제공한다.

