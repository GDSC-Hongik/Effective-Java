## 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

---

### 핵심 정리

- 클래스 내부에서의 자기사용 패턴을 모두 문서로 남기고, 반드시 지켜야 한다
    - 그렇지 않으면, 내부 구현 방식을 활용하던 하위 클래스가 오동작
- 효율 좋은 하위 클래스를 만들 수 있도록, 일부 메서드를 `protected` 제공
    - 클래스 확장할 명확한 근거가 존재하지 않다면, 상속을 금지하기
    - 금지하는 방법은, `final class` 선언, 외부 접근 불가 생성자로 설계

---

## 상속 문서화

> 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.
>
- 재정의 가능 메서드
    - `public` , `protected` 메서드 중 `final` 이 아닌 모든 메서드
- `@implSpec` 태그를 붙이면, 자바독 도구가 생성해준다
- `implementation Requirements(구현 요건)`
    - 메서드의 내부 동작 방식을 설명하는 곳
      
    <img width="520" alt="스크린샷 2023-02-01 오전 5 05 26" src="https://user-images.githubusercontent.com/89639470/215870495-471880b1-3f4c-4b71-8377-039c48c8b1fd.png">
> 좋은 API 문서란 `어떻게` 가 아닌, `무엇` 을 하는지를 설명해야 한다

- 클래스를 안전하게 상속하기 위해선 내부 구현 방식을 설명
    - 상속이 아니었다면 기술하지 않아도 될 사항들

---

## 상속 설계 : Hook 선별

> 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 `protected` 메서드 형태로 공개해야 할 수도 있다

<img width="550" alt="스크린샷 2023-02-01 오전 5 06 04" src="https://user-images.githubusercontent.com/89639470/215870636-4b67f69d-9de6-4dbd-94a8-a0e6a0efaf79.png">

- `clear` 메서드는 `removeRange` 메서드를 호출하여 성능 개선의 목적으로 사용

### Protected 노출할 메서드

> 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 “유일”하다
>
- 필요한 `protected` 멤버를 놓쳤다면, 하위 클래스 작성 시 체감됨
- 반대로, 하위 클래스를 여러 개 만들 때까지 전혀 안 쓰였다면, `private` 할 가능성
    - 이러한 검증에는 하위 클래스 3개를 정도로 테스트가 적당
- **상속용으로 설계한 클래스는 배포 전, 반드시 하위 클래스를 만들어 검증**

> 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다
>
- 상위 클래스 생성자 > 하위 클래스 생성자보다 먼저 실행
    - 하위 클래스에서 재정의한 메서드 > 하위 클래스 생성자보다 먼저 호출
    - 하위 클래스에서 재정의한 메서드가 하위 클래스 생성자에서 초기화하는 값에 의존한다면, 오류 발생

```java
// 상위 클래스
public class Super {
		// 2. 상속용 클래스 생성자에서 재정의 가능 메서드 호출
		// 실제 구현체는 Sub이기에 Null 프린트
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
        System.out.println("Super override method!");
    }
}

// 하위 클래스 정의
public class Sub extends Super {
		// 인스턴스 초기화는 생성자에 맡김
    private final Instant instant;

    public Sub() {
				// 1. 명시적으로 super() 호출하지 않더라도
				// 상위 클래스의 생성자에 기본 생성자 super() 호출
        instant = Instant.now();
    }

		// 재정의 가능 메서드, 상위 클래스의 생성자가 호출한다.
		// 
    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

- `*private` , `final`, `static` 메서드는 재정의가 불가능하니 생성자에서 호출 0*

---

## Cloneable, Serializable 인터페이스

> clone과 readObject 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다
>
- 두 메서드는 생성자와 비슷한 효과(새로운 객체를 생성)

### Cloneable - clone()

- 하위 클래스의 `clone()` 복제본의 상태를 수정하기 전, 재정의한 메서드 호출

### Serializable - readObject()

- 하위 클래스의 상태가 역직렬화되기 전에 재정의한 메서드 호출
- `readResolve()`,`writeReplace()`, `protected` 선언(클래스 API 공개 예시)
    - `private` 선언 시, 하위 클래스에서 무시되기 때문

---

## 상속용 설계 X 클래스

> 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 기본
>
- 클래스에 변화가 생길 때마다 하위 클래스를 오동작하게 만들 수 있기 때문
- 상속을 금지하는 방법
    1. 클래스를 `final` 선언
    2. 모든 생성자를 `private` or `package-private` 선언 후, `public` 정적 팩터리 생성

---

## 상속이 필요하다면

- 이래도, 상속을 꼭 허용해야 한다면 재정의 가능 메서드를 사용하지 않게 만들고 문서로 남기자!
- 다른 메서드의 동작에 아무런 영향을 주지 않을 수 있다
- 클래스 동작을 유지하고 재정의 가능 메서드 사용 코드를 제거하는 방법
    - 재정의 가능 메서드를 `private` 도우미 메서드로 옮기기
    - 도우미 메서드를 호출하도록 수정

```java
// 상속용 클래스
class Super {
    public Super() {
        // overrideMe();
        helpMethod();
    }

    public void overrideMe() {
        helpMethod();
    }

    // 도우미 메서드
    private void helpMethod() {
        System.out.println("Super help Method !");
    }
}

// 하위 클래스
class Sub extends Super {
    private String str;

    public Sub() {
        str = "Sub OverrideMe Method!!";
    }

    @Override
    public void overrideMe() {
        System.out.println(str);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```