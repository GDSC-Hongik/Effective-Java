## 65. 리플렉션보다는 인터페이스를 사용하라

### Reflection

> 구체적인 클래스 타입을 알지 못해도, 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API
>
- 메모리의 힙 영역에 로드된 `Class` 타입의 객체를 사용
- 런타임에 지금 실행되고 있는 클래스에 접근할 경우
    - 동적으로 객체를 생성하고 메서드 호출하는 방법
    - 클래스, 인터페이스, 메서드 찾고 호출할 수 있음

```java
// 리플렉션 실습
class Reflection {
		string name;
	
		Reflection() {
				this.name = "리플렉션";
		}

		Reflection(string name) {
				this.name = name;
		}

		string getName() {
				return this.name;
		}
}

// Constructor
Class clazz = Class.forName("Reflection");
Constructor constructor = clazz.getDeclaredConstructor(); // 인자가 없는 생성자 호출

// Method
Class clazz = Reflection.class;
Method[] methods = clazz.getDelcaredMethods();
System.out.println(methods[0].invoke(clazz.newInstatnce())) // "리플렉션" 출력

// Field
Class clazz = Reflection.class;
Field[] field = class.getDeclaredFields();
System.out.println(field[0]); // out : string Reflection.name

Reflection reflection = new Reflection();
field[0].set(reflection, "Field"); // set 메소드를 사용해서 객체의 변수 변경 가능
System.out.println(field[0].get(reflection)); // "Field" 출력
```

### 단점

> 리플렉션은 아주 제한된 형태로만 사용해야 아래와 같은 단점을 피할 수 있음
>
- 컴파일타임 타입 검사가 주는 이점 상쇄
    - 런타임 오류가 발생하는 경우 존재
- 리플렉션을 이용하면 코드가 장황하고 지저분하다
- 성능 상의 이슈가 존재
    - 일반 메서드 호출보다, 리플렉션을 활용한 메서드 호출이 더 느리다

### 인터페이스 참조 방식

> 리플렉션은 인스턴스 생성에만 쓰고, 인터페이스나 상위 클래스로 참조해 사용하자
>

```java
public static void main(String[] args) {
		// 클래스 이름을 Class 객체로 변환
		Class<? extends Set<Sring>> cl = null;
		try {
				cl = (Class<? extends Set<String>>) // 비검사 형변환 경고
								Class.forName(args[0]);
		} catch (ClassNotFoundException e) {
				fatalError("클래스를 찾을 수 없습니다.");
		}

		// Construtor
		Construtor<? extends Set<String>> cons = null;
		try {
				cons = cl.getDeclaredConstructor();
		} catch (NoSuchMethodException e) {
				fatalError("매개변수 없는 생성자를 찾을 수 없습니다");
		}

		// 집합의 인스턴스
		Set<String> s = null;
		try {
				s = cons.newInstance();
		} catch (IllegalAccessException e) {
        fatalError("생성자 접근 불가");
    } catch (InvocationTargetException e) {
        fatalError("생성자 예외 던짐");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없음");
    } catch (ClassCastException e) {
				fatalError("Set을 구현하지 않음");
		}

		// 생성한 집합 사용
		s.addAll(Arrays.asList(args).subList(1, args.length));
}
```

- 6가지의 예외 처리
    - 리플렉션 없이 진행했다면, 컴파일타임에 잡을 수 있는 코드
- 인스턴스를 생성하기 위해 생성자 호출 1줄이 위와 같이 장황한 코드로 변경

### 리플렉션이 유용한 경우

- 런타임에 존재하지 않을 수 있는 다른 클래스, 메서드, 필드 의존성 관리 목적
    - 여러 개 버전이 존재하는 외부 패키지를 다룰 때
    - 가장 오래된 버전을 컴파일
        - 리플렉션으로 최신 버전으로 접근하면서 새로운 클래스나 메서드에 접근