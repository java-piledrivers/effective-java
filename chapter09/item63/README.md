# 아이템63  문자열 연결은 느리니 주의하라 


- String의 수행 시간은 제곱에 비례해 늘어나고 StringBuilder는 선형으로 늘어난다.

## 문자열 연결 연산자(+)
- 여러 문자열을 하나로 합쳐주는 편리한 수단
- 문자열 n개를 잇는 시간은 n^2에 비례한다.

## StringBuilder
- 성능을 포기하고 싶지 않다면 `StringBuilder`를 사용하자
- StringBuilder.append() 메서드 사용하자


```java
  final long a = System.currentTimeMillis();
        System.out.println(a);
        System.out.println(statement()); //62 ms
        System.out.println(System.currentTimeMillis() - a);

        final long b = System.currentTimeMillis();
        System.out.println(b);
        System.out.println(statement2()); //2 ms
        System.out.println(System.currentTimeMillis() - b);
    }

    private static String statement() {
        String result = "";
        for (int i = 0; i < 10000; i++) {
            result += String.valueOf(i);
        }
        return result;
    }

    private static String statement2() {
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            stringBuilder.append(i);
        }
        return stringBuilder.toString();
    }
```
