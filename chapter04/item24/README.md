# 아이템24 멤버 클래스는 되도록 static으로 만들라

해당 아이템을 이해하기 위해선 먼저 중첩 클래스를 알아야 함

## 중첩 클래스 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

> 참고: [중첩 클래스에 대해 알아보자](https://sjh836.tistory.com/145)

## 정적 멤버 클래스
- 바깥 클래스의 private 멤버에 접근 가능
- private 으로 선언됐을 경우 바깥 클래스에서만 접근 가능
- 크게 일반 클래스랑 다른점은 없음
```java
class Parent {
	private String name;

	static class NestedClass {

	}
}
```
- 주로 헬퍼 클래스로 쓰임
- 146p 3번째 문단
```java
class Main {
	public static void main(String[] args) {
        int result = Calculator.Operation.PLUS.calculate(1, 2);
        System.out.println(result);
   	}
}

class Calculator {
	enum Operation {
	    PLUS, 
	    MINUS, 
	    MULTIPLY, 
	    DIVIDE;

	    int calculate(int x, int y) {
	        switch (this) {
	            case PLUS: return x + y;
	            case MINUS: return x - y;
	            case MULTIPLY: return x * y;
	            case DIVIDE: return x / y;
	        }
	        throw new AssertionError("잘못된 산술" + this);
	    }
	}
}
```

## (비정적) 멤버 클래스
- 정적 멤버 클래스와 구문상 차이점은 단지 static이 있고 없고지만 의미적으론 꽤 큰차이를 가진다
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다.
  - 예제에서 암묵적으로 연결된다고 해놓은 주석 부분이 의문: return this.getName()하면 연결되지 않는거니까
- `ClassName.this`설명: https://docs.oracle.com/javase/specs/jls/se7/html/jls-15.html#jls-15.8.4
  - Qualified `this`
  - https://stackoverflow.com/questions/11276994/what-does-qualified-this-construct-mean-in-java
  ```java
	  class Envelope {
		  void x() {
		    System.out.println("Hello World");
		  }

		  class Enclosure {
		    void x() {
		      Envelope.this.x();
		      // 여기서 this가 Class 이름인 Envelope에 의해서 이 Qualified 되었다고 표현
		      // 이것을 Qualified this (정규화된 this) 라고 표현
		      // Class 이름이 없으면 recursive 되어서 StackOverFlow 발생
		    }
		  }
	  }
  ```

https://javabom.tistory.com/46 에 예제가 있는데 실수했는지..여튼 수정
```java
class Main {
	public static void main(String[] args) {
        NestedNonStaticExample nn = new NestedNonStaticExample("hey");
        System.out.println(nn.getName());
   	}
}

class NestedNonStaticExample {

    private final String name;

    public NestedNonStaticExample(String name) {
        this.name = name;
    }

    public String getName() {
        // 비정적 멤버 클래스와 바깥 클래스의 관계가 확립되는 부분
        NonStaticClass nonStaticClass = new NonStaticClass("nonStatic : ");
        return nonStaticClass.getNameWithOuter();
    }

    public String getA() {
    	return "A";
    }

    private class NonStaticClass {
        private final String nonStaticName;

        public NonStaticClass(String nonStaticName) {
            this.nonStaticName = nonStaticName;
        }

        public String getNameWithOuter() {
            // 정규화된 this 를 이용해서 바깥 클래스의 인스턴스 메서드를 사용할 수 있다.
            return nonStaticName + NestedNonStaticExample.this.getA();
        }
    }
}
```

### 따라서 비정적 멤버 클래스에 static을 붙이자
- 바깥 인스턴스로의 숨은 외부 참조를 가지게 된다.
- GC가 바깥 인스턴스를 수거하지 못하는 경우도 생긴다.

### private 정적 멤버 클래스
- 바깥 클래스가 표현하는 객체의한부분(구성요소)을 나타낼 때 쓴다.
  - 예) Map 구현체의 Entry 객체


## 마무리

**Q) 비정적 멤버 클래스의 주요 역할 - 어뎁터의 역할?**

익명,지역 클래스는 예시 살펴보기
