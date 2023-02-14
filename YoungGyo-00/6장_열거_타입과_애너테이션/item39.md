## 39. 명명 패턴보다 애너테이션을 사용하라

## 명명패턴 단점

```java
// JUnit3 버전까지는 명명패턴 적용
public class TestWorld extends TestCase {
		// 메서드명의 접두사를 test로 시작하게끔
		public void testSafetyOverride() {
				...
		}
}
```

1. 오타가 나면 안됨
    - `tsetSafetyOverride` 로 오타를 내도 JUnit3은 메서드를 무시하고 지나가서 통과한 것처럼
2. 올바른 프로그램 요소에서만 사용된다고 보장 X
    - `TestSafetyMechanisms` 이라고 지어도 JUnit3은 이름에 관심 없음
    - 개발자가 의도한 테스트가 수행되지 않을 수 있음
3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다
    - 특정 예외를 던져야 성공하는 테스트가 있어도, 기대하는 예외 타입을 테스트에 매개변수로 전달해야 함


## 애너테이션(annotation) 선언

- 런타임과 컴파일 시에 해석이 된다
- 애너테이션으로 할 수 있는 일은 명명패턴으로 처리할 수 없음
- 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들을 사용해야 한다

### 메타 애노테이션

```java
// 테스트 메서드임을 선언하는 애너테이션
// 현재는 매개변수가 없는 정적 메서드 전용
@Rentation(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{
}
```

- `@Target`
    - 자바 컴파일러가 애노테이션 어디에 적용될지 결정하기 위해 사용
        - `CONSTRUCTOR` : 생성자 선언 시
        - `METHOD` : 메소드 선언 시
        - `TYPE` : 클래스, 인터페이스, Enum 선언 시
        - …
- `@Rentation`
    - 애노테이션이 실제 적용되고 유지되는 범위를 나타냄
    - `@Rentation(RetentionPolicy.요소)` 형식으로 사용
        - `SOURCE` : 컴파일 이후에도 JVM에 의해서 계속 참조
        - `CLASS` : 컴파일러가 클래스를 참조할 때까지
        - `RUNTIME` : 컴파일 전까지만 유효, 컴파일 후에는 사라짐
- `Documented`
    - 애노테이션이 지정된 대상의 JavaDoc에 애노테이션의 존재를 표기하도록 지정
- `Inherited`
    - 애노테이션을 사용한 상위 클래스를 상속한 하위 클래스에서도 해당 애노테이션을 갖도록 사용

### 마커 애너테이션

- “아무 매개변수 없이 단순히 대상에 마킹(marking) 한다”의 의미
- 애너테이션 사용시, 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 `컴파일 오류`

```java
public class Sample {
		@Test public static void m1()   // 성공
		@Test public static void m2() { // 실패
				throw new RuntimeException("실패"); 
		}
		@Test public void m3() {}       // 잘못 사용 예시
}
```

```java
// 마커 애너테이션 처리 프로그램
public class RunTests {
		public static void main(String[] args) throws Exception {
				int tests = 0;
				int passed = 0;
				// 아직 알려지지 않은 타입(비한정적 타입)
				// 매개변수 타입에 의존하지 않는 제네릭 클래스의 메서드 경우 사용
				Class<?> testClass = Class.forName(args[0]);
				for (Method m : testClass.getDeclaredMethods()) {
						// isAnnotationPresent 가 실행할 메서드를 찾아주는 메서드
						if (m.isAnnotationPresent(Test.class)) {
								tests++;
								try {
                    m.invoke(null);
                    passed++;
											// 테스트 메서드가 예외를 던지면 리플랙션 메커니즘이 감싸서 잡아줌
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause(); // 정보 추출
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
						}
				}
		}
}
```

## 매개변수를 받는 애너테이션

```java
// 매개변수 하나를 받는 애너테이션 타입
@Rentation(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{
		/**
     * Indicates the <em>containing annotation type</em> for the
     * repeatable annotation type.
     * @return the containing annotation type
     */
		Class<? extends Throwable> value(); // value : 매개변수의 타입
}
```

- 특정 예외를 던져야만 성공하는 테스트를 만들기 위해서 `ExceptionTest` 애노테이션을 사용
- 타입이 `Class<? extends Throwable>` 인 매개변수
    - `Throwable` 의 확장한 하위 타입

```java
public class Sample2 {
		// ArithmeticException : 0으로 나누려고 시도할 때
    @ExceptionTest(ArithmeticException.class)
    public static void m1() { // 성공
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() { // 실패 (다른 에러)
        int[] ints = new int[0];
        int i = ints[0];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() {} // 실패 (예외 발생 X)
}
```

```java
// 1개 매개변수를 받는 애너테이션 사용
// 매개변수를 받지 않은 메서드에서 catch 절만 수정
try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
} catch (InvocationTargetException wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    Class<? extends Throwable> excType // 이 부분에서 에러를 보고 기대한 에외가 맞는지
										= m.getAnnotation(ExceptionTest.class).value();
    if (excType.isInstance(exc)) {
        passed++;
    } else {
        System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
    }
} catch (Exception exc) {
    System.out.println("잘못 사용한 @ExceptionTest: " + m);
}
```

- 에러 코드를 테스트하는 것이기 때문에, 필요한 로직이 `catch` 절로 내려옴
- `m.getAnnotation(Exception.class).value()`
    - `Exception.class` 가 붙은 메서드의 매개변수값을 반환
    - 현재 매개변수 값으로 `ArithmeticException` 1개 들어 있음
- 예외의 클래스 파일이 컴파일타임에는 존재했으나, 런타임에는 존재하지 않을 수 있음
    - `TypeNotPresentException` 발생

## 매개변수 2개 이상의 애너테이션

### 1. 배열 매개변수 애너테이션

```java
// 매개변수 여러 개를 받는 애너테이션 타입
@Rentation(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{
		Class<? extends Throwable>[] value();
}
```

```java
// 원소들을 중괄호로 묶고, 쉼표로 구분
@ExceptionTests({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() {
}
```

```java
// 여러 개 매개변수를 받는 애너테이션 사용
// 매개변수를 받지 않은 메서드에서 catch 절만 수정
try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
} catch (InvocationTargetException wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    Class<? extends Throwable>[] excTypes  // 이 부분에서 배열로 매개변수 받기
							= m.getAnnotation(ExceptionTests.class).value();
    for (Class<? extends Throwable> excType : excTypes) {
        if (excType.isInstance(exc)) {
            passed++; // 존재하지 않으면 pass 늘리지 않아서 밑에 if 문 걸리게 됨
            break;
        }
    }
    if (passed == oldPassed) {
        System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
} catch (Exception exc) {
    System.out.println("잘못 사용한 @ExceptionTest: " + m);
}
```

- 반복문을 순회하면서, 예외가 일치하는지 확인

### 2. `@Repeatable` 메타 에너테이션을 활용

- 하나의 프로그램 요소에 여러 번 달 수 있게 해줌
- 주의 사항
    - `@Repeatable` 을 반환하는 “컨테이너 애너테이션”을 하나 더 정의하고 class 객체를 매개변수로 전달
    - “컨테이너 애너테이션”은 내부 애너테이션 타입의 배열을 반환하는 `value` 매서드 정의
    - 적절한 `@Retention`, `@Target` 을 설정
- 사용 이유
    - 별도의 컨테이너 애노테이션을 정의해야 해서 복잡해보이지만, 여러 값을 받아야하는 속성에 따라 하나하나 명시하는 것이 가독성 면에서 좋음

```java
// 실제 사용할 애노테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class) // 묶고자 하는 애너테이션 자체에 선언
public @interface ExceptionTest {
    Class<? extends Throwable> value(); // value 에 들어갈 애노테이션 타입을 중복해서 사용
}

// 애너테이션 묶음 값을 관리할 컨테이너 애노테이션 작성
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

```java
// 중복으로 애노테이션을 달 수 있음
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
}
```

- 주의점
    - 반복 가능 애너테이션을 여러 개 달면, 하나만 달았을 때와 구분하기 위해 컨테이너 애너테이션 타입 적용
    - `getAnnotationByType`
        - 반복 가능 애너테이션, 컨테이너 애너테이션 모두 가져옴
    - `isAnnotationPresent`
        - 반복 가능 애너테이션이 달렸는지 검사하면 의도와 다른 결과 값을 반환
        - 반복 가능 애너테이션과 컨테이너 애너테이션 모두 확이내야 함

```java
//before
if (m.isAnnotationPresent(ExceptionTest.class)) {
		//...
}

//after
if (m.isAnnotationPresent(ExceptionTest.class)
|| m.isAnnotationPresent(ExceptionTestContainer.class)){
		//....
}
```