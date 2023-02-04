[아이템 78] 공유 중인 가변 데이터는 동기화해 사용하라
===

원자성(atomic)이란 메서드나 블록을 한번에 한 스레드식만 실행할 수 있음을 보장함을 말함

동기화를 사용할 경우 변경중인 상태라 일관되지 않은 상태의 객체 값에 접근을 차단함으로써 원자성을 보장한다.

동기화의 또 다른 필요성은 동기화 없이는 한 스레드에서의 변화를 다른 스레드에서 확인할 수 없다.

언어 명세상 long과 double 이외의 변수르 읽고 쓰는 동작은 기본적으로 원자적(atomic) 이다.

-> 항상 어떤 스레드가 정상적으로 저장한값을 온전히 읽어올 수 있음으 보장한다.
하지만 수정이 완전히 반영된 값을 보여줌을 보장하지는 않는다.

##### -> 스레드간 통신에도 동기화가 필요한 이유


## 잘못된 코드

```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        
        
        backgroundThread.start(); //백그라운드 스레드에서 무한루프가 실행됨 
        TimeUnit.SECONDS.sleep(1); 
        stopRequested = true; 
        /* 1초 뒤 stopRequested 를 true로 변경하여 백그라운드 스레드를 멈출려고 하나 
         * 해당 변수를 동기화하지 않았기에, 백그라운드 스레드내에는 변경이 반영되지 않음  
         */
    }
}
``` 

## 수정된 코드
```java

public class StopThread {
    private static boolean stopRequested;

    //변수를 동기화시키는 방법
    //읽기와 쓰기 과정 모두를 동기화시켜야 의도대로 동작한다.
    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

## volatile
배타적 실행이 필요없고, 오직 스레드간 통신만 필요할 경우에 동기화 대용으로 사용을 고려해볼 수 있다.

배타적 수행과는 상관없이 항상 가장 최근에 저장된 값을 읽어온다. 

(이론적으로는 CPU 캐시가 아닌 컴퓨터의 메인 메모리로부터 값을 읽어온다. 그렇기 때문에 읽기/쓰기 모두가 메인 메모리에서 수행된다)

volatile는 안전 실패(safety failure)문제를 발생시킬 수 있음

(이 문제는 메서드에 synchronized를 붙이고 volatile 키워드를 공유 필드에서 제거하면 해결됨) 
// 추가 자료 참조 필요


## atomic 패키지
java.util.concurrent.atomic 패키지에는 락 없이도 thread-safe한 클래스를 제공한다. [아이템 59]

volatile은 동기화의 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다. 게다가 성능도 동기화 버전보다 우수하다.

## 결론

가변 데이터를 공유하지 않는 것이 동기화 문제를 피하는 가장 좋은 방법이다. 
-> 가능하면 가변 데이터는 단일 스레드에서만 사용하도록 설계한다.

한 스레드가 데이터를 수정한 후에 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 
다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication)이라고 한다.

클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드(불변 객체 생성)[item17] 혹은 보통의 락을 통해 접근하는 필드 그리고 동시성 컬렉션에 저장[item81] 하면 안전하게 발행할 수 있다.

        
     
