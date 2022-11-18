[item 35] ordinal 메서드 대신 인스턴스 필드를 사용하라
====
열거 타입에서는 기본적으로 `ordinal`이라는 메서드를 제공합니다. 해당 메서드는 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환합니다.

열거 타입 상수와 연결된 정숫값이 필요하면 사용하고 싶은 유혹에 빠질 수 있습니다. 결론은 **ordinal을 사용하지 말라** 입니다.

### ordinal을 사용한 경우
```java
public enum Ensemble {

    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTEC;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```
책의 예시입니다. 코드를 보면, 다음과 같이 위치와 의미를 일치시킬 수 있습니다. 
ordinal 값은 0부터 시작되는데요, `numberOfMusicians()`메서드에 +1을 해주었습니다. 결과적으로 
혼자 연주하는 SOLO는 1, 둘이서 연주하는 DUET은 2와 자연스레 매칭됩니다.

얼핏 보면 굉장히 좋아보입니다. 따로 무언가를 추가할 필요없이 열거 타입 상수와 연결되어 있고, 의미가 일치하는 정숫값이 자동으로 생성되어 있습니다.
"합주단의 종류"와 "연주자의 수"가 적절히 연관됩니다. 하지만, 동작만 할 뿐 유지보수에 끔찍한 코드입니다.

### 문제 1. 상수 선언 순서 변경
```java
/**
 * 상수 선언 순서를 바꾸는 경우
 * SOLO <-> DUET
 */
enum Ensemble1 {
    DUET, SOLO, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTEC;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```
순서 하나만 실수로 바꾸어도 `numberOfMusicians()`메서드는 오동작 합니다. 실제로 위에 두 enum 클래스들의 같은 의미인 SOLO의 ordinal을 출력하면 다른 값이 나옵니다.
```java
class Main {
    public static void main(String[] args) {
        // 상수 선언 순서를 바꾸는 경우
        System.out.println(Ensemble.SOLO.numberOfMusicians());  // 1
        System.out.println(Ensemble1.SOLO.numberOfMusicians()); // 2
    }
}
```
위의 출력 메서드에서 사용한것처럼 여러 로직들에서 ordinal 값들을 활용한 후, 순서를 바꾸면 프로그램이 엉망이 될 것입니다.

### 문제 2. 이미 쓰이고 있는 정수 값이 같은 상수 추가
```java
/**
 * 8명인 복4중주(double quartet)을 추가하려해도 8이 이미 사용 중
 */
enum Ensemble2 {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, DOUBLE_QUARTET, NONET, DECTEC;

    public int numberOfMusicians() { return ordinal() + 1; }

    public static void main(String[] args) {
        System.out.println(DOUBLE_QUARTET.numberOfMusicians()); // 9
    }
}
```
8명이 연주하는 복4중주(double quartet)도 추가하고 싶지만, 이미 8 정숫값을 OCTEC이 차지하고 있습니다. 정숫값 8위치에 복4중주를 넣을 수가 없습니다.

### 문제 3. 12명이 연주하는 3중 4중주(triple_quartet)를 추가하고 싶다.
```java
/**
 * 중간에 12명이 연주하는 triple quartet을 추가하려는 경우
 * -> 12중주와 숫자 12를 맞추기 위해 억지로 11명짜리 연주를 만들어야 한다. 11명으로 구성된 연주를 일컫는 이름은 없다.
 */
enum Ensemble3 {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTEC, 십일중주, TRIPLE_QUARTET;

    public int numberOfMusicians() { return ordinal() + 1; }
}
```
실제로 11명이 연주하는 연주의 명칭은 없다고 합니다. 하지만 3중 4중주(triple quartet)을 추가하려면 11번째 상수가 필요합니다. 
즉, 쓸데없이 dummy 상수를 추가해야만 합니다. 

코드가 깔끔해지지 못할 뿐 아니라, 쓰이지 않는 값이 많아질수록 실용성이 떨어집니다.

### 결론 : 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하자.
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() { return numberOfMusicians; }
}
```
결국 인스턴스 필드에 직접 저장합니다. 의미가 정확하게 나타나고 유지보수도 편합니다.

애초에 Enum의 API 문서 내용입니다.
> "대부분 프로그래머는 이 메서드를 쓸 일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입기반의 범용 자료구조에 쓺 목적으로 설계되었다."

결국 해당 용도가 아니면 절대 사용해선 안됩니다.