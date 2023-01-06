# 아이템 65. 리플렉션보다는 인터페이스를 사용하라.
리플렉션(java.lang.reflect)를 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.

- 클래스의 생성자, 메서드, 필드에 해당하는 constructor, method, field 인스턴스를 가져올 수 있다.
- 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.
- 위의 것들을 활용해서 실제로 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수도 있다.
- 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.


## 리플렉션 단점 
- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.
- 리플렉션을 이용하면 코드가 지저분해지고 장황해진다.
- 각종 오류 처리로 인해 로직을 보기 힘들며, 코드를 읽기가 힘들어진다.
- 성능이 떨어진다. int 반환 메서드의 경우 일반 메서드에 비해 11배나 느리다.


## 단점을 피하는 방법 
- 리플렉션은 아주 제한된 형태로만 사용한다.
- 리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위클래스로 참조해 사용하자.

## 리플렉션으로 생성하고 인터페이스로 참조해 활용
- 정확한 클래스는 명령줄의 첫 번째 인수로 확정한다. 즉, 컴파일타임에는 어떤 클래스인지 알 수 없다.
- HashSet을 지정하면 무작위 순서가 될 것이고, TreeSet을 지정하면 알파벳 순서가 될 것이다.
```java
public static void main(String[] args) {
    Class<? extends Set<String>> cl = null;
    // 클래스 이름을 클래스 객체로 변환
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]);
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }

    // 생성자를 얻어오기
    Constructor<? extends Set<String>> cons = null;
    try{
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }

    // 위에서 만든 클래스를 통해 집합 인스턴스 생성
    Set<String> s = null;
    try{
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스터화 할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다." + e.getCause());
    } catch (ClassCastException e){
        fatalError("Set을 구현하지 않은 클래스 입니다.");
    }

    // 생성한 인스턴스 사용
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg){
    System.out.println(msg);
    System.exit(1);
}
```

이 예는 리플렉션의 단점을 두가지 보여준다.  
- 런타임 중에 총 여섯가지나 되는 예외를 던질 수 있다. 이 예외들이 인스턴스를 리플렉션 없이 생성했다면 컴파일 타임에 잡아낼 수 있었을 예외들이다.
- 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄 이상의 코드를 작성했다.리플렉션이 아니라면 생성자를 호출하는 한 줄로 끝났을 일이다.   
그래도 위 과정으로 인스턴스를 만들어내면, 이후 사용하는 코드는 기존 인스턴스를 사용할 때와 똑같다.

## 리플렉션 사용 경우
드물긴 하지만, 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다.   
이 기법은 버전이 여러개 존재하는 외부 패키지를 다룰 때 유용하다.    
가동할 수있는 최소환의 환경, 즉 주로 가장 오래된 버전만을 지원하도록 컴파일한 후, 이후 버전의 크래스와 메서드 등은 리플렉션으로 접근하는 방식이다.
