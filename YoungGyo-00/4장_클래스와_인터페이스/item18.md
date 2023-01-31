## 18. 상속보다는 컴포지션을 사용하라

### 핵심 정리

- 이번 아이템에서 정의한 상속은 클래스가 상위 클래스를 확장(`extends`) 하는 상속
- 상속은 캡슐화를 깨뜨리는 문제가 있다
- 상위 클래스와 하위 클래스가 is-a 관계일 때만 사용해야 한다
    - 그래도, 하위 클래스가 상위 클래스와 다르고 상위 클래스가 확장을 고려하여 설계되지 않았다면 문제
- 상속 대신, 컴포지션과 전달(Forwarding)을 사용하자
    - 래퍼 클래스는 하위 클래스보다 견고하고 강력하다

---

## 상속과 컴포지션

```java
class Engine {} // Engine 클래스

class Automobile {} // Automobile 클래스는 Car의 부모 클래스

class Car extends Automobile { // extends -> 상속
	private Engine enigne; // private 로 인스턴스 참조 -> 컴포지션
}
```

- `Car Is-A Automobile` → 상속
- `Automobile HAS-A Engine` → 컴포지션

---

## 상속은 캡슐화를 깨뜨린다!

- 상위 클래스의 구현에 따라, 하위 클래스의 동작에 이상이 생길 수 있음
    - 다른 말로, 구현된 하위 클래스를 수정하지 않고도 상위 클래스의 내부 구현을 바꿨을 때 오동작할 수 있음
- 상위 클래스를 확장을 충분히 고려하고, 문서화 X → 하위 클래스를 수정해야 한다

> 상속을 잘못 구현한 예제

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
	// 추가된 원소의 수
	private int addCount = 0;

	public InstrumentedHashSet() {}

	...코드 생략...
	
	// 이 부분이 중요
	// 문서화가 잘 되어 있지 않은 점
	// HashSet 함수에서 addAll은 add를 호출해서 추가한다.
	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c); // add 함수를 실행해서 추가
	}

	public int getAddCount() {
		return addCount;
	}

	public static void main(String[] args) {
		InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
		s.addAll(List.of("1", "2", "3");
		System.out.println(s.getAddCount()); // 3이아닌 6출력
	}
}
```

### 문제 해결 방법 3가지

> 상위 클래스의 `addAll` 호출하지 않고, 다른 방식으로 재정의하기
>

```java
@Override
public boolean addAll(Collection<? extends E> c) {
	// 이 부분에서 	
	for (E e : c) {
		add(e);
	}

	return ...
}
```

- `HashSet` 의 `addAll` 을 호출하지 않으므로, 정상 작동
- But, 하위 클래스가 깨지기 쉬운 이유 1
    - 메서드 동작을 재구현하는 과정이 어렵고, 오류를 내거나 성능이 저하될 가능성이 높음
    - 추가로, 하위 클래스에서 접근할 수 없게 `private` 필드를 써야 한다면, 구현 자체가 불가능
- 하위 클래스가 깨지기 쉬운 이유 2
    - 상위 클래스에 추가 메서드가 만들어진다면, 하위 클래스에서 재정의하지 못한 그 새로운 메서드를 사용해 ‘허용되지 않은’ 원소를 추가할 수 있는 예시
    - `Hashtable` - `Vector` 컬렉션 프레임워크

> 메서드를 재정의하지 않고, 새로운 메서드를 추가하는 방법
>
1. 상위 클래스에서 추가한 메서드가 하위 클래스에서 새로 추가한 메서드와 시그니처가 같으면, 컴파일 X
2. 하위 클래스에서 구현한 메서드가 상위 클래스의 메서드가 요구하는 규약을 만족시키지 못할 수 있다

> 기존 클래스를 확장하는 대신, 새로운 클래스를 `private` 필드로 기존 클래스의 인스턴스를 참조하게 개발 → 컴포지션 사용
>
- 컴포지션(Composition)
    - 기존 클래스가 새로운 클래스의 구성요소로 설계
    - `private` 필드로 참조하는 인스턴스로 호출
- 전달(Forwarding)
    - 새 클래스의 인스턴스 메서드가 기존 클래스에 대응하는 메서드를 호출하여 반환하는 방식
        - 기존 클래스의 내부 구현 방식의 영향을 받지 않음
        - 새로운 메서드가 추가 되더라도 영향 받지 않음
    - 재사용 가능한 class
- 위임(delegation) = 컴포지션 + 전달 조합의 넓은 의미

```java
// Forwarding class
public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s; // 컴포지션 방식

	public ForwardingSet(Set<E> s) {
		this.s = s;
	}

	// 수많은 메서드 구현...
}

// Wrapper class
public class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;
	
	public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

		public static void main(String[] args) {
				// 다른 Set 인스턴스를 감싸고 있다. 기능을 추가
				Set<Instant> times = new InstrumentedSet<>(new Treeset<>());
		}
}

```

---

## 래퍼 클래스(Wrapper class)

- 기본 타입을 객체화 형태로 포장해서 매개변수로 전달할 때 필요한 클래스
    1. Generic
        - 재사용성을 높이기 위해 클래스 외부에서 정해진 형식에 의존하지 않고, 접근할 때 사용

        ```java
        // ArrayList<T> -> T 부분에 들어가는
        ArrayList<Integer> wrapper = ...
        ```

    2. Casting
        - 기존 타입을 다른 타입으로 변환할 때, 래퍼 클래스의 메소드를 사용

        ```java
        String str = "2";
        int strToint = Integer.parseInt(str);
        ```

- 단점
    - 콜백(callback) 프레임워크와 어울리지 않는다
    - SELF 문제
        - 내부 객체를 감싸는 래퍼의 존재를 모르니, 래퍼가 아닌 내부 객체(this)의 객체를 넘김

        ```java
        // 내부 객체
        public class Model{ 
            Controller controller;
        
            Model(Controller controller){
                this.controller = controller;
        				
        				// Pass SELF reference
                controller.register(this);
            }
        
            public void makeChange(){
                ... 
            }
        } 
        
        public class Controller{
            private final Model model;
        
            public void register(Model model){
                this.model = model;
            }
        
            // 래퍼 클래스의 존재를 모르기 때문에
        		// this(Model)의 makeChange() 실행
            public void doChanges(){
                model.makeChange(); 
            } 
        }
        
        // wrapper class
        public class ModelChangesCounter{
            private final Model; 
            private int changesMade;
        
            ModelWrapper(Model model){
                this.model = model;
            }
        
            // 컨트롤러에서 여기가 호출이 되지 않는 문제   
            public void makeChange(){
                model.makeChange(); 
                changesMade++;
            } 
        }
        ```


---

### 참고자료

- [상속보다는 컴포지션을 사용하라](https://smjeon.dev/etc/composite-extends/)

- [Wrapper Classes are not suited for callback frameworks](https://stackoverflow.com/questions/28254116/wrapper-classes-are-not-suited-for-callback-frameworks)