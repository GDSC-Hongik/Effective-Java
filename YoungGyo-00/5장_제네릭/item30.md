## 30. 이왕이면 제네릭 메서드로 만들라

### 핵심 정리

```markdown
클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환하는 메서드보다 제네릭 메서드가 더 안전
그렇게 하려면 많은 경우, 제네릭 메서드가 되어야 한다
```

### 제네릭 메서드

- 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있음
- 제네릭 메서드 작성법
    - 제네릭 타입 작성법과 비슷

    ```java
    // raw type 사용 - 수용 불가!
    // 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 방법
    // 타입 정보가 전부 지워진 것처럼 사용
    public static Set union(Set s1, Set s2) {
    	Set result = new HashSet<>(s1); // 어떤 타입이 들어갈지 모름 1
    	result.addAll(s2); // result 에 어떤 타입의 s2가 들어갈지 모름 2
    	return result;
    }
    ```

    - 컴파일 O, But 경고 2개

    ```java
    Unchecked call to 'HashSet(Collection<? extends E>)'
     as a member of raw type 'java.util.HashSet' 
    
    Unchecked call to 'addAll(Collection<? extends E>)'
     as a member of raw type 'java.util.Set'
    ```

  [unchecked call to member of raw type](https://stackoverflow.com/questions/38036442/unchecked-call-to-member-of-raw-type)

    - 해결 방법
        - 입력 변수의 원소 타입을 타입 매개변수로 명시
        - 메서드 안에서, 해당 타입 매개변수만 사용하도록 수정
            - **타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다**

        ```java
        // 제너릭 메서드
        public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        	Set<E> result = new HashSet<>(s1); // 어느 타입인지 명확
        	result.addAll(s2); // 같은 타입을 넣음
        	return result;
        }
        
        // 활용
        public static void main(String[] args) {
        	Set<String> guys = Set.of("톰");
        	Set<String> girls = Set.of("래리");
        	Set<String> aflCio = union(guys, girls);
        }
        ```


- 불변 객체를 여러 타입으로 활용할 수 있게 만드는 방법
    - **제네릭은 `런타임` 에 타입 정보가 소거되므로, 하나의 객체를 어떤 타입으로든 매개변수화 가능**
    - 요청한 타입 매개변수에 맞게 객체의 타입을 바꿔주는 정적 팩터리가 필요
        - `제네릭 싱글턴 팩터리`

        ```java
        // 항등함수 예시
        // 비검사 형변환 경고 : casting 할 때 검사를 하지 않았다는 경고
        private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
        
        // 에러, 경고 없이 컴파일 되게 하는 어노테이션
        // UnaryOperator : 입력 받은 매개변수와 리턴 받는 타입이 동일
        @SuppressWarnings("unchecked")
        public static <T> UnaryOperator<T> identityFunction() {
        	return (UnaryOperator<T>) IDENTITY_FN;
        }
        ```

---
### 재귀적 타입 한정(recursive type bound)

- 자기 자신이 들어간 표현식을 사용하여, 타입 매개변수의 허용 범위 제한

```java
// Comparable interface
public interface Comparable<T> {
	int compareTo(T o);
}
```

- `Comparable<T>` 를 구현한 타입이 비교할 수 있는 원소 타입은 거의 자기 자신
    - 제약 코드 예시
    - `<E extends Comparable<E>>` : “모든 타입 `E` 는 자신과 비교할 수 있다”

    ```java
    // 재귀적 타입 한정을 이용해 상호 비교할 수 있음
    public static <E extends Comparable<E>> E max(Collection<E> c);
    ```