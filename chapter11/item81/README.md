# 아이템 81. wait와 notify보다는 동시성 유틸리티를 사용하라

Java에는 동시성을 다루기 위해서 wait와 notify를 지원한다. 다만, 이 메서드들은 Java 5부터 지원하는 다양한 동시성 유틸리티 덕분에 wait와 notify를 사용할 이유가 줄었다.

## wait, notify

### wait
- 갖고 있는 고유 락을 해제하고 스레드를 잠들게하는 메서드.
- 호출하는 메서드가 반드시 고유 락을 가지고 있어야 사용이 가능하다. 즉, synchronized 블록 내부에서 호출이 되어야한다.

### notify
- 잠들어 있던 스레드 중 임의로 하나를 골라 깨우는 메서드. 
- 단, notify는 잠들어 있는 스레드 중 어떤 스레드를 깨울지 선택할 수 없으므로 보통 notifyAll을 사용한다.

### notifyAll

잠들어 있던 스레드 모두를 깨우는 메서드.


## 고수준 동시성 유틸리티
- 실행자 프레임워크
- 동시성 컬렉션 (concurrent collection)
- 동기화 장치 (synchronizer)


## 동시성 컬렉션
- 표준 컬렉션(List, Queue, Map) + 동시성 = 동시성 컬렉션(CopyOnWriteArrayList, ConcurrentHashMap, ConcurrentLinkedQueue)
- 동시성 컬렉션의 동시성을 무력화하는 것은 불가능하며, 외부에서 락을 걸면 오히려 속도가 느려진다. 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출할 수도 없다. 
 
위의 문제를 해결하기 위해 여러 기본 동작들을 하나로 묶는 상태 의존적 메서드가 추가되었다.
```java 
putIfAbsent("key"); // 키에 매핑된 값이 없을 때에만 새 값을 집어넣고, 없으면 그 값을 반환한다.
```

이젠 Collections.synchronizedMap으로 제공되는 SynchronizedMap 보다 ConcurrentHashMap을 사용하는 것이 훨씬 좋다.


## 동기화 장치
동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여 서로 작업을 조율할 수 있도록 해준다. 자주 쓰이는 동기화 장치로는 CountDownLatch와 Semapore가 있으며 CyclicBarrier와 Exchanger, Phaser도 있다.

### CountDownLatch
하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. 생성할 때 int 값을 받는데, 이 값이 countDown()을 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정한다.

```java
public class CountDownLatchTest {
  public static void main(String[] args) {

    ExecutorService executorService = Executors.newFixedThreadPool(10);

    try {
      long result = time(executorService, 5, () -> System.out.println("hello"));
      System.out.println("총 걸린 시간 : " + result);
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      executorService.shutdown();
    }
  }

  public static long time(Executor executor, int concurrency, Runnable action) throws InterruptedException {
    CountDownLatch ready = new CountDownLatch(concurrency);
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(concurrency);

    for (int i = 0; i < concurrency; i++) {
      executor.execute(() -> {
        ready.countDown(); // 타이머에게 준비가 됐음을 알린다.
        try {
          // 모든 작업자 스레드가 준비될 때까지 기다린다.
          start.await();
          action.run();
        } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
        } finally {
          // 타이머에게 작업을 마쳤음을 알린다.
          done.countDown();
        }
      });
    }

    ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
    long startNanos = System.nanoTime();
    start.countDown(); // 작업자들을 깨운다.
    done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
    return System.nanoTime() - startNanos;
  }
}

```

- ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에게 통지할 때 사용한다. 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 기다린다.
- 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start.countDown()을 호출해 기다리던 작업자 스레드들을 깨운다. 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다.
- done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다. 타이머 스레드를 done 래치가 열리자마자 깨어나 종료 시각을 기록한다.
- time메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메서드는 영원히 끝나지 않는다. 스레드의 수가 concurrency 보다 적어 스레드들은 영원히 대기하게 되기 때문이다. 이런 상태를 스레드 기아 교착상태 (thread starvation deadlock)이라고 한다.

# Reference
newFixedThreadPool이란 - https://hamait.tistory.com/937   
세마포어란 - https://lordofkangs.tistory.com/27   
wait와 notify보다는 동시성 유틸리티를 애용하라 - https://velog.io/@chullll/81.-wait%EC%99%80-notify%EB%B3%B4%EB%8B%A4%EB%8A%94-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0%EB%A5%BC-%EC%95%A0%EC%9A%A9%ED%95%98%EB%9D%BC
