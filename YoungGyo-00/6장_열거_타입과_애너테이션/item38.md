## 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

> 열거 패턴은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다
>

### 열거형(Enum)

- 기존의 언어들과 자바의 `Enum` 의 다른 점
    - 열거형이 갖는 값 뿐만 아니라, 타입까지 관리하기 때문에 보다 논리적인 오류를 줄임
    - `타입에 안전한 열거형` 을 제공하여 실제 값이 같아도 타입이 다르면 `false`
        - 값 뿐만 아니라 타입까지 체크
    - 상수 값이 바뀌어도 기존 소스를 다시 컴파일하지 않아도 된다


### 타입 안전 열거 패턴(typesafe enum pattern)

- 클래스를 이용하고, 생성자를 `private` 로 만들어 최초 정의된 객체만을 참조
- 타입 안전 패턴은 확장할 수 있으나, 열거 타입은 불가능하다는 점 하나를 제외하면 열거 타입이 우수

```java
public class TypesafeOperation {
		private final String type;
		// Enum의 생성자가 private 인 이유?
		// 고정된 상수의 집합으로 런타임이 아닌, 컴파일 타임에 모든 값을 알고 있어야 하기 때문에
		// 즉, 다른 패키지나 클래스에 접근해 동적으로 값을 할당할 수 없음
		private TypesafeOperation(String type) {
				this.type = type;		
		}
		
		public static final TypesafeOperation PLUS = new TypesafeOperation("+");
		...
}
```

### 인터페이스를 이용한 확장 가능 열거 타입 흉내

- 확장할 수 있는 열거 타입이 어울리는 쓰임 → 연산 코드(Operation code)
    - 연산 코드용 인터페이스를 정의
    - 열거 타입이 이 연산 코드용 인터페이스를 구현
- API  가 제공하는 연산 이외의 확장 연산을 추가할 수 있도록 열어줌

```java
// 인터페이스를 통해 확장 가능 열거 타입을 흉내
// 연산을 수행하라는 간단한 apply 메서드만 정의
public interface Operation {
		double apply(double x, double y);
}

// 각각 apply 메서드 구현
public enum BasicOperation implements Operation {
		PLUS("+") {
				public double apply(double x, double y) {return x + y;}
		}
		MINUS ...
		TIMES ...
		...

		private final String symbol;
	
		BasicOperation(String symbol) {
				this.symbol = symbol;
		}
	
		@Override public String toString() {
				return symbol;
		}
}
```

- `BasicOperation` 은 확장할 수 없지만, `Operation` 인터페이스를 확장할 수 있음
    - 인터페이스를 연산의 타입으로 사용
- 연산 타입을 확장한 지수 연산과 나머지 연산을 추가해보자

```java
// 인터페이스를 확장하여 추가 연산 만들기
public enum ExtendedOperation implements Operation {
		EXP("^") {
				public double apply(double x, double ) { return Math.pow(x, y);}
		}
		REMAINDER ...

		private final String symbol;
		...
}
```

- `apply` 가 인터페이스에 선언되어 있기 때문에, 추상 메서드를 선언하지 않아도 됨
- 기존 연산을 쓰던 곳에, `ExtendedOperation` 을 사용할 수 있음

```java
public static void main(String[] args) {
		double x = Double.parseDouble(args[0]);
		double y = Double.parseDouble(args[1]);
		test(ExtendedOperation.class, x, y);
}

// 제네릭 T는 Enum, Operation을 확장한(하위 타입)
// Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻
// 열거 타입이어야지 순회할 수 있고, Operation이어야지 원소 연산을 할 수 있음
private static <T extends Enum<T> & Operation> void Test (
				Class<T> opEnumType, double x, double y) {
		for (Operation op : opEnumType.getEnumConstants())
				System.out.printf("%f %s %f = %f%n",
											x, op, y, op.apply(x,y));
}
```

- `<T extends Enum<T> & Operation>`
    - Class 객체가 열거타입인 동시에 Operation 인터페이스의 구현체여야 한다

```java
// 두 번째 방법은 Class 객체 대신 한정적 와일드카드 타입 Collection<? extends Operation>을 넘김
public static void main(String[] args) {
		...
		test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
		...
}
```

- 인터페이스를 이용한 확장 가능 열거 타입의 문제점
    - 열거 타입끼리 구현을 상속할 수 없음
    - 연산 기호를 찾고 저장하는 로직이 중복
- 중복량이 적으니 문제 없지만, 공유하는 기능이 많다면 별도의 도우미 클래스 혹은 정적 도우미 메서드로 분리