## 55. 옵셔널 반환은 신중히 하라

### JAVA 8 이전 방법

- 특정 조건에서 값을 반환할 수 없을 경우
    - 예외 던지기
        - 진짜 예외적인 상황에서만 사용
        - 예외 생성 시, 스택 추적 전체를 캡처하므로 비용적인 측면에서 부담
    - NULL 반환
        - NULL 처리 코드 작성
        - `NullPointerException` 이 엉뚱한 곳에 발생할 수 있음

### JAVA 8 이후

- `Optional<T>`
    - `T` 타입 하나를 담았다 or 아무것도 담지 않았다라고 표현
    - 보통 `T` 를 반환해야 하지만, 아무것도 반환하지 않을 때 사용

```java
// 컬렉션에서 최댓값을 구해 Optional<E> 로 반환 코드
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		if (c.isEmpty())
				return Optional.empty(); // 정적 팩터리를 사용해 옵셔널 생성

		E result = null;
		for (E e: c)
				if (result == null || e.compareTo(result) > 0)
						result = Objects.requireNonNull(e);

		return Optional.of(result);// 값이 든 옵셔널
}

// 스트림 버전
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		return c.stream().max(Comparator.naturalOrder());
}
```

- `Optional.of(value)`
    - `null` 값을 넣으면 `NullPointerExcepion` 에러  출력
    - `Optional.ofNullable(value)` 사용하길
        - NULL인지 아닌지 확신할 수 없는 객체를 담고 있는 Optional 객체를 생성

### Optional 반환 선택 기준

> 옵셔널은 검사 예외와 취지가 비슷하다
>
- 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다
    - 수행 중에 일어날 수 있는 예외를 미리 검사하고 대비하려는 목적
- 검사 예외를 던지면 클라이언트에서 값을 받지 못할 때 행동을 코드로 작성
    - 옵셔널을 반환할 때, 동일하게 캐치하는 코드 작성

    ```java
    // 기본값을 정할 수 있다
    // 비어있는 객체에 대해서 넘어온 인자를 반환
    String lastWordInLexicon = max(word).orElse("단어 없음");
    
    // 원하는 예외를 던질 수 있다
    // 비어있는 Optional 객체에 대해서 생성된 예외를 던짐
    Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
    
    // 항상 값이 채워져 있다고 가정
    // 확신할 수 있다면 곧바로 꺼내서 사용할 수 있음
    Element lastNobleGas = max(Elements.NOBLE_GASES).get();
    ```


### isPresent Method

- 옵셔널이 채워져 있으면, `True` 아니면 `False`
    - 더 짧고 명확한 용법에 맞는 코드 사용 가능

    ```java
    // 부모 프로세스의 프로세스 ID 혹은 "N/A" 출력
    Optional<ProcessHandle> parentProcess = ph.parent();
    
    System.out.println("부모 PID:" + (parentProcess.isPresent() ?
    		String.valueOf(parentProcess.get().pid()) : "N/A")); // 삼항 연산자 사용
    
    // map 사용
    System.out.println("부모 PID:" +
    		ph.parent().map(h-> String.valueOf(h.pid))).orElse("N/A");
    
    // filter 사용
    streamOfOptionals
    		.filter(Optional::isPresnet)
    		.map(Optional::get)
    
    // stream flatMap()
    streamOfOptionals
    		.flatMap(Optional::stream)
    ```


### Optional 이 독이 되는 경우

> 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안 된다
>
- 빈 컨테이너를 그대로 반환 시, 옵셔널 처리 코드를 넣지 않아도 되기 때문
    - `Optional<List<T>>` 를 반환하기 보다, `List<T>` 를 반환

### Optional 을 사용하는 경우

> 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리할 경우
>
- `Optional` 도 새로 할당하고 초기화하는 객체, 그 값을 꺼내기 위하여 한번 더 메서드 호출
    - 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있음

> 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자
>
- 옵셔널 전용 클래스들이 존재
    - `OptionalInt`, `OptionalLong`, `OptionalDouble`

> 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다
>
- 옵셔널을 맵의 값으로 사용하면 안 된다