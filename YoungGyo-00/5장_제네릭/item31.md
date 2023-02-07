## 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

### 핵심 정리

```java
와일드카드 타입을 적용하면, API가 훨씬 유연
널리 쓰일 라이브러리를 작성한다면, 와일드카드 타입을 적절히 사용
PECS - Comparable, Comparator 모두 소비자
```

### 불공변 & 와일드카드

> `List<String>` 은 `List<Object>` 는 서로 다르다

- 제네릭은 기본적으로 불공변
    - 하위 타입도, 상위 타입도 아니다
- 매개변수화 타입은 아이템 28에서 이야기했듯 불공변
- 대신 `와일드카드` 를 적용하면, 공변과 반공변으로 표현이 가능
- Stack Class 예시)

    ```java
    public class Stack<E> {
    	public Stack();
    	public void push(E e);
    	public E pop();
    	public boolean isEmpty();
    	// 메서드 추가
    	// 일련의 워너소를 스택에 넣는 메서드
    	public void pushAll(Iterable<E> src) {
    		for (E e : src)
    			push(e);
    	}
    }
    ```

- 논리적으로 동작 가능해 보이는 코드
    - `Number` 는 기본적으로 `Integer` 보다 상위 클래스

    ```java
    Stack<Number> numberStack = new Stack<>();
    Iterable<Integer> intergers = ...;
    numberStack.pushAll(intergers); // 불공변이기에 오류
    ```


> 유연성을 극대화하려면, 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.


### 상한 경계 와일드 카드

- `<? extend T>`
    - 들어오는 타입의 경우 `T` 의 자식 객체 타입
- `E의 Iterable` 이 아닌, `E의 하위 타입의 Iterable` 로 수정

```java
// E 생산자(producer) 매개변수에 와일드카드 타입 적용
public void pushAll(Iterable<? extends E> src) {
	for (E e : src) 
		push(e);
}
```

### 하한 경계 와일드 카드

- `<? super T>`
    - 들어오는 타입의 경우 `T` or 부모 객체 타입
- `popAll` 메서드 구현 시, 똑같은 문제 발생
    - 이번엔 `E의 상위 타입의 Collection` 로 작성

    ```java
    // E 소비자(consumer) 매개변수에 와일드카드 타입 적용
    public void popAll(Collection<? super E> dst) {
    	while(!isEmpty())
    		dst.add(pop());
    }
    ```


### 펙스(PECS) : producer-extends, consumer-super

> 반환 타입은 여전히 Set<E>임에 주목. 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다.


```java
// 30-2 코드 예시)
public static <E> Set<E> union(Set<E> s1, Set<E> s2)

// PECS 변경 후
public static <E> Set<E> union(Set<? extend E> s1, Set<? extends E> s2)
```

> `Comparable<E>` , `Comparator<E>` 는 항상 소비자


```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

- 전체적인 플로우
    - `E` 타입을 List 인자로 받은 후, 받은 List 중에서 가장 큰 값을 리턴하는 메서드
    - `return` 값은 `comparable`
- 입력 매개변수
    - `E` 인스턴스를 생산하므로, `extends`
- 타입 매개변수
    - `Comparable` 은 항상 소비자의 역할이므로, `super`

> `Comparable` 을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드


```java
public interface Comparable<E>
public interface Delayed extends Comparable<Delayed>
public interface ScheduledFuture<V> extends Delayed, Future<V>
```

> 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체


```java
// swap 두 가지 선언
// 1. 비한정적 타입 매개변수
public static <E> void swap(List<E> list, int i, int j);
// 2. 비한정적 와일드카드
public static void swap(List<?> list, int i, int j);
```

- `Public API` 라면 간단한 두 번째 방법을 사용
    - 하지만 직관적으로 구현한 코드가 컴파일 안 되는 문제

        ```java
        public static void swap(List<?> list, int i, int j) {
        	list.set(i, list.set(j, list.get(i)));
        }
        ```

    - 리스트 타입이 List<?> 인데, 이는 `null` 외에는 어떤 값도 넣을 수 없다
    - `private 도우미 메서드` 를 따로 작성하여 활용
        - 제네릭 메서드로 구현

        ```java
        public static void swap(List<?> list, int i, int j) {
        	swapHelper(list, i, j);
        }
        
        // 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
        // 방법 1과 메서드 시그니처가 동일
        public static <E> void swapHelper(List<E> list, int i, int j) {
        	list.set(i, list.set(j, list.get(i)));
        }
        ```