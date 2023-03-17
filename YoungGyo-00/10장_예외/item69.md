## 69. 예외는 진짜 예외 상황에서만 사용하라

> 예외는 정상적인 제어 흐름에서 사용하면 X, 프로그래머에게 강요하는 API X


```java
// 배열의 끝에 도달해 예외가 발생하면 마무리
try {
		int i = 0;
		while (true)
				range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {

}

// 표준적인 관용구로 작성 시
for (Mountain m : range)
		m.climb();
```

- 예외를 사용한 이유
    - 배열에 접근할 때마다 경계를 넘지 않는지 검사 → 반복문 또한 배열 경계에 도달하면 종료
    - 반복문에 명시하면, 같은 일이 중복되므로 하나를 생략
        - 중복되는 연산을 줄여 성능을 높이려는 전략

### 3가지 잘못된 추론

> 에외를 사용한 쪽이 표준 관용구보다 훨씬 느리다

1. 예외는 예외 상황에 사용될 용도로 설계
    - JVM 구현차 입장에서는 명확한 검사만큼 빠르게 만들어야할 동기가 약하다
2. 코드를 `try~catch` 블록 안에 넣으면, JVM 이 적용할 수 있는 최적화가 제한됨
3. 배열을 순회하는 표준 관용구는 중복 검사를 수행하지 않음
    - JVM이 알아서 최적화해 없애줌

- 결론적으로, 반복문 안의 버그를 숨겨버리고, `ArrayIndexOutOfBoundsException` 에러가 다른 이유로 발생했을 경우에도 오류가 나지 않고 정상적으로 반복문을 종료

### 상태 검사 메서드

> 잘 설계된 API라면, 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없게 제작

- 상태 의존적 메서드 : `next`
- 상태 검사 메서드 : `hashNext`

```java
// hasNext(상태 검사 메서드)를 제공하지 않았을 때
try {
		Iterator<Test> i = collection.iterator();

		while (true) {
				Test test = i.next();
				...
		}
} catch (NoSuchElementException e) {

}

// Iterator 표준 관용구 사용
for (Iterator<Test> i = collection.iterator(); i.hashNext();) {
		Test test = i.next();
		...
}
```

### 옵셔널 사용

- 외부 동기화 없이 여러 스레드가 동시 접근 가능하거나, 상태가 변할 수 있다면 옵셔널을 사용
- 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행한다면 옵셔널
- 그 외의 경우 상태 검사 메서드 방식이 더 낫다