

## 요약
제어 흐름을 컨트롤하려는 목적으로 try-catch를 사용하지마라

- 2배 정보 느렸다? -> 실제로 해보니 둘다 비슷함.
	- 내가 테스트를 잘못했을까? 맨아래 테스트코드 넣음.
- 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 해야한다.
	- 상태 의존적 메소드를 제공하는 클래스는 '**상태 검사' 메서드도 함께 제공**해야한다
		- Iterator
			- next, hasNext
	- 상태 의존 메소드를 사용한 덕분에 for 문을 사용할 수 있다.
		- for-each 도 내부적으로 hasNext 를 사용한다
```java
for (Iterator<Foo> i = collection.iteratior(); i.hasNext();) {
	Foo foo = i.next();
	...
}
```
- Iterator 가 hasNext 를 제공하지 않으면 그 일을 클라이언트가 대신해야만 했다
```java
try {
	Iterator<Foo> i = collection.iteratior();
	while(true) {
		Foo foo = i.next();
		...
	}
} catch (NoSuchElementException e) {

}
```
- 반복문 예외 사용하면 속도느리다, 장황, 헷갈, 엉뚱한곳에서 발생한 버그를 숨기기도 함


## 다른 선택지
- 상태 검사 메서드 대신 사용할수 있는지 선택지
	- 옵셔널, 특정값 (388p)
		- https://johngrib.github.io/wiki/java/exception-handling/#from-%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C-%EC%9E%90%EB%B0%94
	- 옵셔널 경우는 여러스래드가 동시접근하거나 외부요인으로 상태변할수있다면 옵셔널이나 특정값을 사용한다
		- 상태 의존적 메서드 호출하는 와중에 상태가 변할 수 있기 때문


## 테스트 코드

```java
import java.util.Date;  
import java.util.UUID;  
  
public class TestMain {  
    public static void main(String[] args) {  
        long startTime = new Date().getTime();  
        Range[] range = init();  
  
        // v1  
        //  v1(range);  
        // v2      
        //  v2(range);  
  
        long diff = new Date().getTime() - startTime;  
        System.out.println(diff);  
    }  
  
    private static void v2(Range[] range) {  
        for (Range r : range) {  
            r.climb();  
        }  
    }  
  
    private static void v1(Range[] range) {  
        try {  
            int i = 0;  
            while (true) {  
                range[i++].climb();  
  
            }  
        } catch (ArrayIndexOutOfBoundsException e) {  
            logger.error(e.getMessage(), e, e.getClass());  
            System.out.println(e);  
        }  
    }  
  
    public static Range[] init() {  
        int FIX_VAL = 10000000;  
        Range[] range = new Range[FIX_VAL];  
        for (int i = 0; i < FIX_VAL; i++) {  
            range[i] = new Range();  
        }  
        return range;  
    }  
  
    public static class Range {  
        public void climb() {  
            UUID uuid = UUID.randomUUID();  
        }  
    }  
}
```
