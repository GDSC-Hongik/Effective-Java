# Item 21 : 인터페이스는 구현하는 쪽을 생각해 설계해라

## 핵심정리

- 기존 인터페이스에 디폴트 메서드 구현을 추가하는 것은 위험한 일이다.
- 디폴트 메서드는 구현 클래스에 대해 아무것도 모른 채 합의 없이 무작정 "삽입" 될뿐이다
- 디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬 수 있다.
- 인터페이스를 설계할 때는 세심한 주의를 기울여야한다.
- 서로 다른 방식으로 최소한 세가지 구현을 해보자.

### 인터페이스의 디폴트 메서드 추가?

```java
// collection.java 의 디폴트 메서드.
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

```

synchronizedCollection 입장에서는..
멀티스레드에서 접근할 때 안전하게 접근할 수 있는 자료구조인데,
저 removeif 때문에 안전하질 못함.

```java
//synchronized collection
// 결국 컴파일 에러가아닌 이렇게 오버라이딩을 따로 직접 해줘야함.
// 예상치 못한 동작을 일으킬 수 있음!
        @Override
        public boolean removeIf(Predicate<? super E> filter) {
            synchronized (mutex) {return c.removeIf(filter);}
        }

```

### 순위 문제

```java
//SuperClass
public class SuperClass {

    private void hello() { // private 임
        System.out.println("hello class");
    }
}


//MarkerInterface
public interface MarkerInterface {

    default void hello() {
        System.out.println("hello interface");
    }

}
// Main
public class SubClass extends SuperClass implements MarkerInterface {

    public static void main(String[] args) {
        SubClass subClass = new SubClass();
        // 나이거 쓸 수 있다고 표시됨!!!
        // 그러나 SuperClass hello는 private 이기 때문에 IllegalAccessError 발생.
        subClass.hello();
    }
}
// 에러는 컴파일 에러가 제일 좋다~
```

### ConcurrentModificationException

- 멀티스레드가 아니라 싱글 스레드 상황에서도 발생 할 수 있다. 가령 fail-fast 이터레이터를 사용해
  콜렉션을 순회하는 중에 콜렉션을 변경하는 경우.

```java
public class FailFast {

    public static void main(String[] args) {
        // 이건 불편 콜렉션 이에요
       List<Integer> numbers = List.of(1, 2, 3, 4, 5);

        // 변경 가능한 컬렉션.
        List<Integer> numbers = new ArrayList<>();
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
        numbers.add(4);
        numbers.add(5);

        // 이터레이터로 콜렉션을 순회하는 중에 Collection의 remove를 사용한다면...
       for (Integer number : numbers) {
           if (number == 3) {
               numbers.remove(number);
           }
       }

        // 이터레이터의 remove 사용하기 이터레이터가 알고 있으니 괜찮음.
       for (Iterator<Integer> iterator = numbers.iterator(); iterator.hasNext();) {
           Integer integer = iterator.next();
           if(integer == 3) {
               iterator.remove();
           }
       }

       // 인덱스 사용하기 예외가 터짐.
       for (int i = 0; i < numbers.size() ; i++) {
           if (numbers.get(i) == 3) {
               numbers.remove(numbers.get(i));
           }
       }

        // removeIf 사용하기
       numbers.removeIf(number -> number == 3);

        // 출력
        numbers.forEach(System.out::println);
    }
}


```
