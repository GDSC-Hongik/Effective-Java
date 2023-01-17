## 1. 생성자 대신 정적 팩터리 메서드를 고려하라

---

```markdown
핵심 정리
- 정적 팩터리 메서드와 public 생성자 각자의 상대적인 장단점 존재
- 정적 팩터리 메서드 사용이 유리한 경우가 더 많음
  장점
    1. 이름을 가질 수 있다.
    2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
    3. 반환 타입의 하위 타입 객체를 반환할 능력이 있다.
    4. 입력 매개변수에 따라 다른 클래스 객체를 반환할 수 있다.
    5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
  단점
    1. 상속하려면 public, protected 생성자가 필요하니, 정적 팩터리 메서드만 제공할 시 하위 클래스를 만들 수 없다.
    2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
```

### 매서드 시그니처

- 메서드 명과 파라미터의 순서, 타입, 개수를 의미
- No! 리턴 타입, Exceptions

```java
// 서로 다른 시그니처
void doSomething(String[] x); // doSomething(String[]) - 메서드 시그니처 예 1
void doSomething(String x); // doSomething(String)

// 같은 시그니처
int doSomething(int x); // doSomething(int)
void doSomething(int y) throws Exception; // doSomething(int)
```

[Type signature - Wikipedia](https://en.wikipedia.org/wiki/Type_signature)

### 정적 팩토리 메서드

- 객체의 생성을 담당하는 클래스 메서드
- `new` 키워드를 클라이언트 코드에서 직접 선언하지 않고, 정적 팩토리 메서드 내에서 간접적으로 사용

```java
// String.java valueOf
public static String valueOf(@NotNull char data[]) {
	return new String(data);
}

// public 생성자와 정적 팩토리 메서드 비교
public static void main(String[] args) {
	String str1 = String.valueOf("정적 팩터리 메서드");
	String str2 = new String("pullic 생성자");
}
```

- 장점
    - 이름을 가질 수 있다 (동일한 시그니처의 생성자를 두개 가질 수 없다)
        - `public` 생성자보다 객체의 특성을 제대로 설명
        - 한 클래스에 시그니처가 같은 생성자를 여러 개 생성 가능
    - 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다
        - 불변 클래스(immutable class)는 `Instance` 를 재활용 → 불필요한 객체 생성 X
        - 인스턴스 통제 가능 → 싱글턴으로 만들 수 있고, 인스턴스화 불가로 만들 수 있음
    - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
        - 객체 생성 시, 분기처리를 통해 하위 타입의 객체를 반환할 수 있음

        ```java
        public class Grade {
        	...
        	private static Grade of(int semester) {
        		if(0 < semester && semester <= 2) {
        			return new Freshman();
        		}
        		if(2 < semester && semester <= 4) {
        			return new Sophomore();
        		}
        		...
        	}
        }
        ```

      [정적 팩토리 메서드(Static Factory Method)](https://velog.io/@cjh8746/%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9CStatic-Factory-Method)

    - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**(EnumSet)**

      ### EnumSet

        - enum과 함께 동작하는 `Set`
        - EnumSet은 다른 Collection과 다르게 `new` 연산자로 사용 불가
        - 정적 팩토리 메소드만으로 EnumSet의 구현 객체를 반환 받을 수 있음

        ```java
        // EnumSet 내부 코드
        public abstract class EnumSet {
          ...
          // noneOf 메소드에서 원소의 갯수 64를 기준으로 다른 구현체 객체 반환
          public static noneOf(...) {
        		if (enumElementSize <= 64) 
        			return new RegularEnumSet<>(); // 원소 갯수가 적을 때 적합
        		else
        			return new JumboEnumSet<>(); // 원소 갯수가 많을 때 적합
        	}
        }
        
        // 서적에 있는 예시 기준 생성하는 방법
        public abstract class Pizza {
        	public enum Topping { HAM, MUSHROOM, ONION ... }
        	
        	abstract static class Builder<T extends Builder<T>> {
        		// 비어있는 EnumSet을 만들어 반환
        		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        		// 모든 요소들을 포함하는 EnumSet
        		EnumSet<Topping> toppings = EnumSet.allOf(Topping.class);
        	}
        }
        ```

      [EnumSet이 new 연산자를 사용하지 않는 이유](https://siyoon210.tistory.com/152)

    - 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

- 단점
    - `public` , `protected` 없이, 정적 팩터리 메서드만 제공하면 상속 X → 하위 클래스를 만들 수 없다
    - 정적 팩토리 메서드는 프로그래머가 찾기 어렵다
        - 생성자처럼 API 설명에 명확히 드러나지 않기 때문에, 인스턴스화할 방법을 알아야 한다.