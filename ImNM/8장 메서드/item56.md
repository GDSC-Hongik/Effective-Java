# Item 56 : 공개된 api 요소에는 항상 문서화 주석을 작성하라

## 핵심정리

- 여러분의 api를 올바로 문서화하려면 공개된 모든 클래스 , 인터페이스 , 메서드 , 필드 선언에 문서화 주석을 달아야 한다.
- 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명로하게 기술
- 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야한다.
- 클래스 혹은 정적 매서드가 스레드 안전하든 그렇지 않든 스레드 안전 수준을 표시
- 자바독 유틸리티를 적극 활용해라!

### 1. 여러분의 api를 올바로 문서화하려면 공개된 모든 클래스 , 인터페이스 , 메서드 , 필드 선언에 문서화 주석을 달아야 한다.

잘 작성된 문서도 곁들여야 API를 쓸모있게 할 수 있다.

자바에서는 자바독 이라는 유틸리티가 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API문서로 변환

공개된 API는 무조건 달아라!

상속용으로 설계된 클래스의 메서드가 아니라면
how 가 아닌 what을 기술. 즉 무엇을 하는 메서드인지.

또한 사후 조건 도 고려

- @throws
  비검사 예외를 선언하여 암시적으로 기술
- @param
  조건에 영향받는 매개변수에 기술
- @return
  반환타입이 void가 아니라면
- @code
  태그로 감싼 내용을 코드용 폰트로 렌더링
  태그로 감싼 내용에 포함된 HTM 요소나 다른 자바독 태그를 무시한다.

```java
   /**
     * Returns a BigInteger whose value is equivalent to this BigInteger
     * with the designated bit cleared.
     * (Computes {@code (this & ~(1<<n))}.)
     *
     * @param  n index of bit to clear.
     * @return {@code this & ~(1<<n)}
     * @throws ArithmeticException {@code n} is negative.
     */
    public BigInteger clearBit(int n) {
```

- @impleSpec
  해당 메서드와 하위 클래스 사이의 계약을 설명
  하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지
  사용하도록 해줘야함.

- @literal
  특수문자 집어넣고싶을때 사용

- @summary
  요약 설명 용
- @index
  색인화 가능

```java
    /**
     * Returns a sequential {@code Stream} with this collection as its source.
     *
     * <p>This method should be overridden when the {@link #spliterator()}
     * method cannot return a spliterator that is {@code IMMUTABLE},
     * {@code CONCURRENT}, or <em>late-binding</em>. (See {@link #spliterator()}
     * for details.)
     *
     * @implSpec
     * The default implementation creates a sequential {@code Stream} from the
     * collection's {@code Spliterator}.
     *
     * @return a sequential {@code Stream} over the elements in this collection
     * @since 1.8
     */
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```

### 2. 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야한다.

```java
/**
 * <p>This interface is a member of the
 * <a href="{@docRoot}/java.base/java/util/package-summary.html#CollectionsFramework">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see HashMap
 * @see TreeMap
 * @see Hashtable
 * @see SortedMap
 * @see Collection
 * @see Set
 * @since 1.2
 */
public interface Map<K, V> {
```

그외
열거타입 문서화 할때는 상수에도 주석
애너테이션 타입을 문서화 할때는 멤버들에도 모두 주석
