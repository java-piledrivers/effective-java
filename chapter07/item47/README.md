
## 정리

Stream 보다 Collection 을 리턴타입으로 사용하라

Why?

- Collection 은 iterator() 나 stream() 메소드를 지원한다.
	- stream 으로 처리하거나 for-each 로 처리하길  원하는 사용자 모두 만족시킨다
		- Stream 을 for-each 에서 사용하기 위해선 어댑터나 캐스팅이 필요하다
- Collection 과 Stream 인터페이스가 iterator 메소드를 가지고 있긴 하지만, Collection 만 Iterable 을 extend 하기 때문이다.
	- iterable 을 extend 한다는 의미는 무엇인가?
		- An iterable interface **allows an object to be the target of [enhanced for loop](https://www.geeksforgeeks.org/for-each-loop-in-java/)**(for-each loop).
		- 참고: [Iterable 란 무엇인가?](https://www.geeksforgeeks.org/iterable-interface-in-java/)

## 코드 예시

- [링크](https://sejoung.github.io/2019/02/2019-02-08-Prefer_Collection_to_Stream_as_a_return_type/#%EC%95%84%EC%9D%B4%ED%85%9C-47-%EB%B0%98%ED%99%98%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C-%EC%8A%A4%ED%8A%B8%EB%A6%BC%EB%B3%B4%EB%8B%A4-%EC%BB%AC%EB%A0%89%EC%85%98%EC%9D%B4-%EB%82%AB%EB%8B%A4)

## 왜  Stream 은  Iterable 을 extend 하지 않나??

- Iterable 은 재사용성을 암시하지만, Stream 은 한번 반복문을 돌리는 "반복자에"에 더 가깝다.
- 참고: (https://stackoverflow.com/a/20130131
