``` java

public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

``` java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}

```

``` java

class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}

```


한 파일에 톱레벨 클래스를 2개이상씩 담을경우 소스 컴파일을 어떻게 하냐에 따라 프로그램 실행 결과가 달라진다.

혹은 컴파일 결과가 예상과 다르게 나올 수 있다.



해결방법:

파일 하나당 톱레벨 클래스는 하나씩 생성해야한다.
