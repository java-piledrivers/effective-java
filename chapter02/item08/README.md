자바에서 비메모리자원회수는 try-with-resources, try-finally 로 한다. 사실 try문을 사용하지 않아도 자바에선 자동으로 내부적으로 gc에 의해 finalize()를 통해 가져가긴 하지만 명시적으로 try문을 사용해 꼭 close 해주어야하긴 한다.

finzlize() 를 여기서 finalizer 라고한다. finalizer와 cleaner는 반드시 사용하지 말아야한다. 이펙티브자바에서는 써도 되는 때에 대해서 2가지 소개를 하긴 하지만 쓰지말아야하는 이유가 압도적으로 많고 그 이유를 분명히하고 있다. 때문에 사용하지 말아야한다.

finalizer를 사용하지 말아야하는 이유는 다음과 같다.

1. **즉시 수행된다는 보장이 없다.** finalizer와 cleaner는 호출된 후에도 언제 실행될지 알 수가없다. 다른 메소드들처럼 바로 실행되는 게 아니라 이상하게도 finalizer는 바로 실행되지가 않아서이다. finalizer와 cleaner는 수행 속도는 GC 구현마다 다르다. GC가 실행될 떄 finalize()가 GC 대상인데 언제 호출될진 모른다.

2. **자원 회수가 지연된다.**

3. **심지어 수행 여부조차 보장되지 않는다.**
사실 앞에서 gc의 작업대상이 되서 finalize()가 실행된다곤 했지만 이것을 보장하진않는다. 보장해주는 System 패키지에 runFinnalizersOnExit와 Runtime 패키지에 runFinalizersOnexit 를 통해 보장해주긴 하지만 매우 치명적 결함이 있으므로 사용하지말자

4. **동작중 발생한 예외가 무시된다.**
finalizer는 동작 중 발생할 예외를 무시하고 처리할 작업이 있더라도 그 순간 종료된다.

5. **가비지 컬렉터 효율을 떨어뜨린다**

6. **보안 문제를 일으킬 수 있다**
https://github.com/java-piledrivers/effective-java/issues/8
