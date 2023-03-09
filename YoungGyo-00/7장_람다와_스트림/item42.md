## 42. 익명 클래스보다는 람다를 사용하라

### 익명 클래스

- 함수 객체를 만드는 주요 수단
    - 추상 메서드를 하나만 담은 인터페이스의 인스턴스

    ```java
    // 낡은 기법 - 익명 클래스의 인스턴스를 함수 객체로 사용
    Collections.sort(words, new Comparator<String>() {
    		public int compare(String s1, String s2) {
    				return Integer.compare(s1.length(), s2.length());
    		}
    });
    ```

    - 문자열을 길이순으로 정렬하는 비교 익명 클래스 사용
    - `Comparator` 인터페이스가 정렬을 담당하는 추상 전략을 의미
        - 문자열을 정렬하는 구체적인 전략을 `익명 클래스`로 구현
        - 코드가 너무 길기 때문에, 자바는 함수형 프로그래밍에 적합하지 않음

### 람다식(Lambda)

> 타입을 명시해야 코드가 더 명확할 때를 제외하고, 람다의 모든 매개변수 타입은 생략하자


```java
// 람다식을 함수 객체로 사용 : 익명 클래스 대체
Collections.sort(words,
				(s1, s2) -> Integer.compare(s1.length(), s2.length()));
				// 람다 : Comparator<String>
				// s1 return type : String
				// s2 return type : String
				// 반환값 : Int
```

- 컴파일러가 타입을 추론해줘서, 타입을 명시하지 않음
- 컴파일러가 타입 추론을 실패했다는 오류가 발생할 때만, 타입을 명시

```java
// 람다 자리에, 비교자 생성 메서드를 사용하면 더 간단하게
Collections.sort(words, comparingInt(String::length);
```

```java
// List 인터페이스에 추가된 sort 메서드를 이용하면 더 짧아짐
words.sort(comparingInt(String::length));
```

### Item 34 예제 변형

```java
// 기존 코드
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

```java
public enum Operation {
	PLUS("+") {"+", (x, y) -> x + y}
	...

	private final String symbol;
	private final DoubleBinaryOperator op;
	
	Operation(String symbol, DoubleBinaryOperator op) {
			this.symbol = symbol;
			this.op = op;
	}
	...
	public double apply(double x, double y) {
		return op.applyAsDouble(x, y);
	}
}
```

- 열거 타입 상수의 동작을 표현한 람다 `DoubleBinaryOperator` 인터페이스 변수에 할당
- `double` 타입 인수 2개를 받아 `double` 타입 결과 반환

### 람다 기반 Operation 열거 타입

- 상수별 클래스 몸체는 더 이상 사용할 이유가 없다고 느낌
- 하지만, 람다에도 단점이 존재
    - 이름이 없고, 문서화도 못 함
    - 코드 자체로 동작이 명확히 설명되지 않거나, 코드 줄 수가 많아지면 람다 사용 자제
- 추상 클래스의 인스턴스를 만들 대 람다를 사용할 수 없음
    - 함수형 인터페이스에서만 람다가 사용되기 때문
    - 이 때는, 람다 대신 익명 클래스를 사용