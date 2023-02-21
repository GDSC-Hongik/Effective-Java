# Item 47 : 반환 타입으로는 스트림보다 컬렉션이 낫다

## **스트림 Streams**

자바 8에서 추가한 스트림(*Streams*)은 람다를 활용할 수 있는 기술 중 하나. 자바 8 이전에는 배열 또는 컬렉션 인스턴스를 다루는 방법은 `for` 또는 `foreach` 문을 돌면서 요소 하나씩을 꺼내서 다루는 것이 일반적이었음. 로직이 복잡해질 수록 이해하기 어려울정도로 for문이 많아지는 것을 방지하기 위해 나온 개념.

### 스트림과 컬렉션의 비교

int 형태의 배열을 가지고 중복을 제거하고 내림차순으로 정렬한 뒤 List 형태로 반환하는 자바 코드를 stream으로 구현한 코드를 각각 컬렉션과 스트림으로 비교해보자

- 컬렉션

```java
int[] arr = {5, 3, 1, 4, 5, 2, 2, 1};
List<Integer> result = new ArrayList<>();

// 중복 제거
for (int num : arr) {
    if (!result.contains(num)) {
        result.add(num);
    }
}

// 내림차순 정렬
Collections.sort(result, Collections.reverseOrder());
```

- 스트림

```java
int[] arr = {5, 3, 1, 4, 5, 2, 2, 1};
        
List<Integer> result = Arrays.stream(arr)
        .distinct()
        .boxed()
        .sorted(Collections.reverseOrder())
        .collect(Collectors.toList());

System.out.println(result);
```

단, Array는 스트림을 가지고 있으니까 

```java
int[] arr = {5, 3, 1, 4, 5, 2, 2, 1};
List<Integer> result = new ArrayList<>();

// 중복 제거
Arrays.stream(arr).distinct().forEach(result::add);

// 내림차순 정렬
Collections.sort(result, Collections.reverseOrder());

System.out.println(result);
```

요롷게 바꿀 수 있다. 

보는 것처럼 컬렉션이나 array는 stream을 가지고 있어서 왠만하면 컬렉션을 넘겨주는 것이 좋은 방법이라고 할 수 있겠다~

문제는 `stream`에는 `foreach`를 사용하지 못해서 불편한 경우가 있다는것.

아래 두 예시처럼

어댑터 메서드를 사용하여 스트림을 iterate로 변경하거나

```java
//Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
```

반대로 API가 Iterable을 받아서 스트림을 반환하는 어댑터로 구현가능하다

```java
//Iterabel<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

동작은 하지만 복잡하고 직관성이 떨어지지.

사용자의 입장에서 스트림과 Iterable중 어느 것을 사용할 지 모르기에 한 쪽만 가능한 publid API 범용성이 떨어지기에 둘 다 지원되도록 하는 쪽이 나은데. 

Collection은 Iterable 하위 타입이고, Stream 메서드도 지원한다. 그 예로 Arrays 역시 asList와 Stream.of 메서드로 쉽게 반복문과 Stream을 지원할 수 있다. **공개 API의 반환 타입에는 왠만해선 둘 다 가능한 컬렉션이나 그 하위 타입을 쓰는게 보통 최선**이다.

단 그렇다고 **아무런 생각없이 무조건 대용량의 시퀀스를 메모리에 올려선 안된다.** 

예를 들어, 주어진 집합의 멱집합을 반환하는 상황이라고 가정해보자.

{a, b, c}의 멱집합은 { {}, {a}, {b}, {c}, {a, b}, {a, c}, {b, c}, {a, b, c} }이다. 멱집합의 원소의 수는 2^3개 이다. 만약 원소 수가 N개라면 멱집합의 원소 개소는 2^n개가 된다. 만약 집합의 원소가 20개라면 이 컬렉션은 2^20만큼의 멱집합 원소를 탐색 후 반환해야 한다.

따라서 컬렉션의 contains와 size를 시퀀스의 내용을 확정하기 전 까지 구할 수 없는 경우(i.e. 실제 반복을 돌려보기 전 까지는 무엇이 얼마나 들어갈지 예측이 불가능한 경우)에는 Stream을 반환하는 것이 좋다.

### 핵심정리

- 원소 시퀀스를 반환하는 메서드를 작성할 때는 시퀀스와 스트림으로 모두 처리하기 원하는 사용자들을 고려하자.
- 그래서 가능하면 컬렉션을 반환하는 것이 좋다.
- 반환 전에도 원소들을 컬렉션에 담고 있거나 숫자가 아주 작으면 ArrayList도 좋다.
- 컬렉션이 만약 불가능하면 stream과 Iterable중 더 자연스러운걸 반환하라