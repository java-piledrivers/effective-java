
# 아이템54 null이 아닌, 빈 컬렉션이나 배열을 반환하라

- 타이틀 그대로 컬렉션이나 배열이 비어있을 때 null 을 반환하지마라.
	- null 반환시 방어코드 작성 필요
	- 빈 컨테이너 할당도 비용이 든다?
		- 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한 이 정도의 성능 차이는 신경 쓸 수준이 아니다
	- 배열도 마찬가지로 빈배열을 반환하라
- 사실 주변에서 흔히 볼 수 있는 코드라면서 처음에 코드를 보여주는데...왜 이렇게 작성하는지 도저히 의도를 모르겠다.
	- 코드54-1 에서 저렇게 하기보다는 내가 저런 의도의 코드를 작성한다면 그냥 `cheeseInStock` 의 getter 을 사용할 것o 같다
	- 그리고 new ArrayList 담아서 해야하나?
```java
private final List<Cheese> cheeseInStock = ...;

public List<Cheese> getCheeses() {
	return cheesesInStock.isEmpty() ? null : new ArrayList<>(chesseInStock);
}
```


**참고** 

cheeseInStock 을 new ArrayList 해서 내보내는 이유?
- 방어적복사를 다시 보자 
