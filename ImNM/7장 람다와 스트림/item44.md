# Item 44 : 표준 함수형 인터페이스를 사용하라

## 핵심정리

- 필요한 용도에 맞는게 있다면 , 직접 구현하지 말고 표준 함수형 인터페이스를 사용하라!!
- 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자 ( 아이템 61 )
  박싱 int Integer 같은거
- 직접 만든 함수형 인터페이스에는 항상 @FuncationalInterface 사용하자.

### 1. 직접 구현하지 말고 표준 함수형 인터페이스를 사용하라.

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```

```java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

잘 동작하긴 하지만 자바 표준 라이브러리에 준비되어있음!!

BiPredicate<Map<K,V> , Map.Entry<K,V>>

java.util.function 에는 총 43개의 인터페이스가 담겨있음

### 2. 기본형 6가지

- UnaryOperator<T> : T apply(T t)
- BinaryOperator<T> : T apply(T t1,T t2)
- Predicate<T> : boolean test(T t)
- Function<T,R> : R apply(T t)
- Supplier<T> : T get()
- Consumer<T> : void accept(T t)

```java
String str;
void a(Consumer<String> func){
    Objects.requireNotNull(func);
    // 어떤걸 실행
    func.accept(str);
}
```

### 3. 직접 구현은?

- 자주 쓰이며 , 이름 자체가 용도를 명확히 설명해준다.
- 반드시 따라야 하는 규약이 있다.
- 유용한 디폴트 메서드를 제공할 수 있다.

@Override 를 사용하는거와 마찬가지처럼
함수형 인터페이스라는 것을 명시 할 수 있는
@FuncationalInterface 를 직접 만든 함수형 인터페이스에 사용하라.

- 해당 클래스의 코드나 설명 문서에 람다용 으로 설계된 것을 알수 있음.
- 해당 인터페이스가 추상 메서드 오직 하나만 가지고 있어야 컴파일 되게 해준다.
- 유지 보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
