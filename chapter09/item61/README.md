# 아이템 61 박싱된 기본 타입보다는 기본 타입을 사용하라

- 박싱된 것과 아닌 것은 오토박싱/언박싱 덕분에 구분을 하지 않고 사용할 수 있다.
- 그렇다고 성능적으로도 차이가 없다는 것은 아니다.

```java
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++)
		sum += i;

	return sum;
}
```

무슨 차이가 있는가?

1. 식별성의 유무
2. 유효성 - 박싱된 값은 null을 가질 수 있다.
3. 효율성 -  시간과 메모리 사용면에서 뭐가 더 효율적일까?

1번 식별성의 유무에 대한 예제 코드. 무엇이 문제인가?
```java
Comparator<Integer> naturalOrder =
	(i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

2번 유효성에 대한 예제 코드. 무엇이 문제인가?
```java
public class Unbelivable {
	static Integer i;
	
	public static void main(String[] args) {
		if (i ==42)
			System.out.println("Unbelivable!");
	}
}
```

3번 효율성에 대한 예제 코드. 무엇이 문제인가?
```java
private static long sum() {
	Long sum = 0L;
	for (long i = 0; i <= Integer.MAX_VALUE; i++)
		sum += i;

	return sum;
}
```

- 결론, 가능하면 기본 타입을 선택해라.
	- 그렇다면 왜 굳이 박싱된 기본 타입을 사용하는가?
		- 제네릭에 넣어서 사용못한다. 가령 컬렉션같은.
			- [더 자세하게 읽어보기](https://stackoverflow.com/a/27647772)
