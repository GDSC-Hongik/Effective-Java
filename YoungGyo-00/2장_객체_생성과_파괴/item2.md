## 2. 생성자에 매개변수가 많다면 빌더를 고려하라

---

```markdown
핵심 정리
- 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면, 빌더 패턴을 선택
- 매개변수 중 다수가 필수가 아니거나 같은 타입이면 더욱 활용
- 생성자 패턴보다 코드 읽기 수월, 자바빈즈보다 훨씬 안전
```

### 생성자 패턴

- 확장하기 어렵다
- 매개변수가 많아지면, 클라이언트 코드 작성하기 어렵다

```java
public class NutritionFacts {
	private final int servingsSize; // 필수
	private final int servings; // 필수
	private final int fat; // 선택
	...

	public NuritionFacts(int servingSize, int servings) {
		this(servingSize,servings,0);
	}

	public NuritionFacts(int servingSize, int servings, int fat) {
		this(servingSize,servings,fat, 0);
	}
	
	public NuritionFacts(int servingSize, int servings, int fat, ...) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.fat = fat;
		...
	}
}

// 클라이언트 코드 예시 - 매개변수를 잘못친 에러를 찾아내기 힘들다
NutritionFacts food = new NutritionFacts(240, 80);
```

### 자바빈즈 패턴

- **객체 하나를 만들려면 메서드를 여러 개 호출**
- **완전히 생성되기 전까지 일관성(consistency)이 무너진 상태**
- **클래스를 불편으로 만들 수 없음**

```java
public class NutritionFacts {
	// 매개변수 생략

	// No 매개변수 생성자
	public NutritionFacts() {}
	
	// 수많은 세터(Setter)
	public void setServingSize(int val) { servingSize = val; }
	...
}
```

### Builder 패턴

- 점층적 생성자 패턴의 안정성 + 자바빈즈 패턴의 가독성
- **필요한 매개변수만으로 생성자를 호출**해 빌더 객체 생성
- 생성할 클래스 안에 **정적 멤버 클래스**로 만들어두는 게 일반적이다
    - **플루어트 API(fluent API)** - 물 흐르듯 연결된다는 의미
    - **메서드 연쇄(method chaining)**

```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int fat;
	...

	// 정적 멤버 클래스로 생성할 클래스 안에 빌더 만들기
	public static class Builder {
		// 필수 매개변수
		private final int servingSize;
		private final int servings;

		// 선택 매개변수
		private int fat = 0;
		...

		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder fat(int val) {
			fat = val; return this;
		}
		...
		
		// 마지막에 매개변수 없는 build 메서드를 호출하여 객체 생성
		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}
	
	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		fat = builder.fat;
		...
	}
} 

// 클라이언트 코드
NutritionFacts food = new NutritionFacts.Builder(240,8)
								.fat(50).build();
```

- **빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다**
    - 가변인 매개변수를 여러 개 사용할 수 있다는 장점
    - 하단의 `addTopping` 메서드 참고

```java
// 루트 추상 클래스
public abstract class Pizza {
	public enum Topping { HAM, ... }
	final Set<Topping> toppings;

	// 추상 클래스는 추상 빌더를 가짐. 하위 클래스에서 구체 빌더로 구현
	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping)); // Null 체크
	    return self();
    }
		
		abstract Pizza build();

		// 하위 클래스에서 메소드 재정의하여 "this"를 반환하게 해야 함.
    protected abstract T self();
	}

  Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone(); // 복사본 만들기
	}
}

// 뉴욕 피자
public class NyPizza extends Pizza {
	public enum Size { SMALL, ... }
	private final Size size;

	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;
		
		public Builder(Size size) {
			// 잘못된 매개변수 확인하는 용도 -> invariant(불변식) 만족하는 조건
			this.size = Objects.requireNonNull(size);
		}

		@Override public NyPizza build() {
			return new NyPizza(this);
		}

		@Override protected Builder self() { return this; }
	}

	private NyPizza(Builder builder) {
		super(builder);
		size = builder.size;
	}
}

// 클라이언트 코드
NyPizza pizza = new NyPizza(SMALL)
						.addTopping(SAUSAGE).addTopping(ONION).build();
```