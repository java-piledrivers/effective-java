[아이템 78] 공유 중인 가변 데이터는 동기화해 사용하라
===

원자성(atomic)이란 메서드나 블록을 한번에 한 스레드식만 실행할 수 있음을 보장함을 말함

동기화를 사용할 경우 객체나 변수가 변경중인 상태라 일관되지 않은 상태일때 접근을 차단함으로써 원자성을 보장한다.

또하 동기화가 없다면, 한 스레드에서의 변화를 다른 스레드에서 확인하지 못할 수 있다.

언어 명세상 long과 double 이외의 변수르 읽고 쓰는 동작은 기본적으로 원자적(atomic) 이다.
(long과 double은 프로세서에서 64비트 연산이 지원되지 않아, 32비트로 나누어 연산을 진행하도록 설계 되었다고함 - 해당 내용은 후에 추가적인 보충 필요) 


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


MultiThread 어플리케이션에서는 기본적으로 Task를 수행하는 동안 성능 향상을 위해 Main Memory에서 읽은 변수 값을 CPU Cache에 저장한다.
그렇기에 Multi Thread 환경에서 Thread가 변수 값을 읽어올 때는 각각의 CPU Cache에 저장된 값이 다르기 때문에 변수 값 불일치 문제가 발생하게 된다.

<img width="541" alt="스크린샷 2023-02-05 오전 10 48 49" src="https://user-images.githubusercontent.com/30370933/216797529-3bb501cc-3e47-4e95-b404-bc4199f7afa0.png">


이때 배타적 실행이 필요없고, 오직 스레드간 통신만 필요할 경우에 volatile을 동기화 대용으로 사용을 고려해볼 수 있다.

volatile은 배타적 수행과는 상관없이 항상 가장 최근에 저장된 값을 읽어온다. 

volatile은 매번 변수의 값을 Read, Write 작업 모두 CPU cache에 저장된 값이 아닌 Main Memory에서 읽어 오기 때문에 변수 값 불일치 문제을 해결할 수 있다.


단 volatile는 안전 실패(safety failure)문제를 발생시킬 수 있음
모든 스레드가 동시에 하나의 변수에 접근해 값을 Write할 경우 역시 변수값이 일치하지 않을 위험이 있음 

volatile을 사용한 스레드간 통신은 Multi Thread 환경에서 단 하나의 Thread만 read & write하고 나머지 Thread들은 read하는 상황에서 사용하는게 최선이라고 생각됨

(여러 스레드가 가변 데이터에 write해야한다면 volatile 키워드 대신 동기화를 사용해야함) 


https://nesoy.github.io/articles/2018-06/Java-volatile


## atomic 패키지
java.util.concurrent.atomic 패키지에는 락 없이도 thread-safe한 클래스를 제공한다. [아이템 59]

volatile은 동기화의 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다. 게다가 성능도 동기화 버전보다 우수하다.

## 결론

처음부터 가변 데이터를 공유하지 않는 것이 동기화 문제를 피하는 가장 좋은 방법이다. 
-> 가능하면 가변 데이터는 단일 스레드에서만 사용하도록 설계한다.

한 스레드가 데이터를 수정한 후에 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 
다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication)이라고 한다.

클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드(불변 객체 생성)[item17] 혹은 보통의 락을 통해 접근하는 필드 그리고 동시성 컬렉션에 저장[item81] 하면 안전하게 발행할 수 있다.

        
     
