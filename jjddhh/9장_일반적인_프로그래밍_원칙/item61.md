# 박싱된 기본 타입보다는 기본 타입을 사용하라

기본 타입과 박싱된 기본 타입의 주된 차이는 세 가지다.
- 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성이란 속성을 갖는다. 
- 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값, 즉 null을 가질 수 있다.
- 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

위의 세 가지 차이 때문에 주의하지 않으면 문제가 발생할 수 있다.

### 문제들

```java
Comparator<Integer> naturalOrder =
        (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
이 코드는 대부분의 경우에는 정상적으로 값을 반환한다.

하지만 `naturalOrder.compare(new Integer(42), new Integer(42))` 이 코드는 어떨까?

메서드 반환값으로 0 을 기대하겠지만 1 이 나온다.

이는 부등호 연산의 경우 객체들을 오토언박싱 해주지만, == 연산의 경우 양쪽이 객채면 언박싱 없이 레퍼런스 값을 비교하게 된다.

그래서 결과적으로 0 이 아닌 1 이 나오게 된 것이다.

다음 문제되는 코드를 보자.
```java
public class Unbelievable {
    static Integer i;
    
    public static void main(String[] args) {
        if (i == 42)
            System.out.println("믿을 수 없군!");
    }
}
```
`(i == 42)` 가 참이 아니라는건 자명하다. 하지만 어떤 결과가 나올 것인지는 예상하기 힘들 것이다.

해당 코드는 NullPointerException 을 던진다. 

그 과정을 설명해보면,`(i == 42)` 연산에서 i 를 언박싱한다. 

여기서 i 는 현재 null 값을 가지고 있고 null 을 언박싱하려는 결과로 NullPointerException 이 발생하는 것이다. 


```java
public static void main(String[] args) {
        Long sum = 0L;
        for(long i = 0; i <= Integer.MAX_VALUE; i++) {
            sum += i;
        }
}
```
이 코드는 sum 이 Long 으로 선언되었기 때문에 의미없는 오토박싱 계속 발생하기에 성능저하가 일어났다.

그럼 박싱된 기본 타입은 언제 써야 하는가?

컬렉션의 원소, 키, 값으로 사용하면 된다. (ex. ThreadLocal<Integer>)
