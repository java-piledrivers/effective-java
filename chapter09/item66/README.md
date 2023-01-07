### **자바 네이티브 인터페이스(JNI)란?**

자바 프로그램이나, C, C++ 같은 네이티브 프로그래밍 언어로 작성된 네이티브 메서드를 말한다. 

---

### **JNI의 주요 쓰임 3가지**

첫번째, 레지스트리 같은 플랫폼 특화 기능을 사용한다. 

두번째, 네이티브 코드로 작성된 기존 라이브러리를 사용한다.  

세 번째, 성능 개선을 목적으로 성능에 결정적인 영향을 주는 영역만 따로 네이티브 언어로 작성한다.

> 성능을 개선할 목적으로 네이티브 메서드를 사용하는 것은 거의 권장하지 않는다. ( 이제 충분히 JVM 발전했기 때문)

생각보다 네이티브 메서드가 성능을 개선해 주는 일이 많지 않다.
> 

---

### **JNI 단점.**

1) 메모리 훼손 오류. 

- 네이티브 언어가 안전하지 않으므로 **네이티브 메서드를 사용하는 어플리케이션도 메모리 훼손** 오류로부터 더 이상 안전하지 않다.
- 가비지 컬렉터가 네이티브 **메모리는 자동 회수하지 못하고 심지어 추적**조차 할 수 없다.

2) 이식성

- 네이티브 언어는 자바보다 플랫폼을 많이 타서 이식성도 낮고 디버깅도 더 어렵다.

3) 성능과 비용, 가독성

- 주의하지 않으면 속도가 오히려 느려질 수 있다.
- 자바 코드와 네이티브 코드의 경계를 넘나들 때마다 **비용도 추가된다.**
- 네이티브 메서드와 자바 코드 사이의 **접착 코드(glue code)’**를 작성해야 하는데, 이는 귀찮은 작업이거나 가독성도 떨어진다.

### 핵심 정리

- 네이티브 메서드를 사용하려거든 한번 더 생각하라. 네이티브 메서드가 성능을 개선해주는 일은 많지 않다.
- 저수준 자원이나 네이티브 라이브러리를 사용해야만 해서 어쩔 수 없더라도 네이티브 코드는 최소한만 사용하고 철저히 테스트하라.
- 네이티브 코드 안에 숨은 단 하나의 버그가 어플리케이션 전체를 훼손할 수 있으므로 주의하라.

---

- [Java Native Memory Leak 원인을 찾아서. - Toss](https://www.google.com/search?q=what+is+native+moery+%2F+native+method&oq=what+is+native+moery+%2F+native+method&aqs=chrome..69i57j33i160l2.26863j0j7&sourceid=chrome&ie=UTF-8#fpstate=ive&vld=cid:29954d01,vid:w4fWgLgop5U)
    
    Java Native Memory Leak 예상 범위 찾기. 
    
    1. JNI & JNA 
        
        Java Native Interface & Java Native Access
        
        c로 구현한 모듈을 자바에서 직접호출하는 기능. 
        
        c구현체에서 메모리 릭이 생길 경우 자바 가상 머신에서는 이를 인지 할 수 없어 문제가 생길 수 있다.
        
    2. Direct Buffer
        
        JDK 1.4부터 지원하는 DirectBuffer
        
        DirectBuffer 버퍼로 메모리를 할당하게 되면 자바 가상머신의 gc로 관리되지 않는 네이티브 메모리가 할당된다. 
        
    
    1. JavaAgent 기반의 APM 툴. 
        
        Application Performance Managemnt
        
        APM 툴은 보통 JavaAgent로 연동하며 instrument를 변경하는 형태로 동작하니 거기로 혹시 모를 네이티브 메모리를 쓰는 부분은 없을까?
        
    
    ---
    
    Process search ? 
    
    C2 Compiler ? JIT Compiler?
    
    C1 Compiler? C2 Compiler? 
    
    Graal Compiler ?
