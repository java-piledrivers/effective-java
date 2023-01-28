[아이템 72] 표준 예외를 사용하라
===

자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.

## 표준 예외를 사용할 경우 장점
- API가 다른 사람이 익히고 사용하기 쉬워진다.
- 해당 API를 이용한 프로그램도 낯선 예외를 사용하지 않게 되어 읽기 쉽게 된다.
- 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다.

## 가장 널리 재사용되는 예외들
| 예외                              | 주요쓰임                                                      |
|---------------------------------|-----------------------------------------------------------|
| IllegalArgumentException        | 허용하지 않는 값이 인수로 건네졌을 때(null은 따로 NullPointerException으로 처리) |
| IllegalStateException           | 객체가 메서드를 수행하기에 적절하지 않은 상태일  때                             |
| NullPointerException            | null을 허용하지 않는 메서드에 null을 건넸을 때                            |
| IndexOutOfBoundsExcpetion       | 인덱스가 범위를 넘어섰을 때                                           |
| ConcurrentModificationException | 허용하지 않는 동시 수정이 발견됐을 때                                     |
| UnsupportedOperationException   | 호출한 메서드를 지원하지 않을 때                                        |

### 상황에 부합한다면 항상 표준 예외를 사용하자.
- 위 표의 내용이 가장 확실히 흔히 사용되는 예외들이다.다른 상황에서 다른 예외도 재사용 가능하다. ArithmeticException, NumberFormatException...
- <U>API 문서를 참고해 그 예외가 어떤 상황에 던져지는지 꼭 확인해야 한다. 예외의 이름뿐 아니라 예외가 던져지는 맥락도 부합할 때만 사용한다.</U>

### Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.
- 추상 클래스라고 생각하길 바란다.
- 여러 성격의 예외들을 포괄하는 클래스이므로 안정적으로 테스트할 수 없다.

### IllegalStateException vs IllegalArgumentException
- 앞서 표의 주요 쓰임이 상호 베타적이지 않아서 위처럼 종종 재사용할 예외를 선택하기 어렵다.
- 인수의 값이 무엇이었든 어짜피 실패했을 거라면 IllegalStateException 던지기
- 그렇지 않으면 IllegalArgumentException을 던지기