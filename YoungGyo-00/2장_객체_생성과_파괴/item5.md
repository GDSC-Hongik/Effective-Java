## 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

---

```markdown
핵심 정리
- 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면,
싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다.
- 클래스를 직접 만들게 해서도 안 된다.
- 대신 필요한 자원을 생성자에게 넘겨주는 의존 객체 주입 기법을 사용
 - 자원을 만들기 위한 팩터리를 생성자(혹은 정적 팩터리나 빌더)에 넘겨주는 것
```

### 자원을 직접 명시

- 유연하지 않고 테스트하기 어렵다
- 동작이 주입 자원에 따라 변할 수 있으므로, 코드를 계속 변경하며 재사용이 불가능하다
- 테스트에 효육적이지 않다

```java
public class SpellChecker {
	// 사전에 따라 우측 ADictionary() 부분이 바뀔 수 있음
	private static final Dictionary dictionary = new ADictionary(); 
}
```
라
### 의존 객체 주입

- `Dictionary` 가 변경되어도 `SpellCheck` 의 코드를 재사용할 수 있다

```java
// 유연한 방식
public class spellChecker {
	private final Dictionary dictionary;
	
	public SpellChecker(Dictionary dictionary) {
		// 주입하는 객체가 빈값이 아님을 필터링
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public static boolean isValid(String word){...}

  // functions
}
```

- 테스트용 주입도 가능하다

    ```java
    class SpellCheckerTest {
    	@Test
    	void isValid() {
    		SpellChecker spellChecker = new SpellChecker(new MockDictionary());
    		assertTrue(SpellChecker.isVaild("test");
    	}
    }
    ```

- 정적 팩터리를 넘겨주는 방식

    ```java
    public static SpellChecker staticFactory(Dictionary dictionary) {
    	return new SpellChecker(dictionary); // 유연하게 주입 가능
    }
    ```

- 빌더에 적용

    ```java
    public static Builder builder(Dictionary dictionary) {
    	return new Builder(dictionary);
    }
    
    public static class Builder{
    	private Dictionary dictionary;
    	
    	public Builder(Dictionary dictionary) {
    		this.dictionary = dictionary;
    	}
    }
    ```


### 팩터리 메서드 패턴

- 객체를 생성할 때, 어떤 클래스의 인스턴스를 만들지 서브 클래스에서 결정하도록 위임
- 부모 추상 클래스 : 인터페이스에만 의존
- 서브 클래스 : 어떤 구현 클래스를 호출할지 결정
    - 새로운 구현 클래스가 추가되어도 기존 Factory 코드의 수정 없이 새로운 Factory 추가 가능

  ![스크린샷 2023-01-17 오후 5 42 27](https://user-images.githubusercontent.com/89639470/212850117-5d91d40a-b092-457e-9bcc-0ebe9e834c77.png)
    - 사용자 관리 프로그램이 네이버 계정으로 가입할 수 있는 코드

    ```java
    // Product
    public interface User {
    	void signup();
    }
    
    // ConcreteProduct
    public class NaverUesr implements User {
    	@Override
    	public void signup() {
    		System.out.println("네이버 아이디로 가입");
    	}
    }
    
    // Creator
    public abstract class UserFactory {
    	public User newInstance() {
    		User user = createUser();
    		user.signup();
    		return user;
    	}
    
    	protected abstract User createUser();
    }
    
    // ConcreteCreator
    public class NaverUserFacotry extends UserFactory {
    	@Override
    	protected User createUser() {
    		return new NaverUser();
    	}
    }
    ```

    ```java
    // ConcreateProduct(Kakao 버전)
    public class KakaoUser implements User {
        @Override
        public void signup() {
            System.out.println("카카오 아이디로 가입");
        }
    }
    
    // ConcreateCreator(Kakao 버전)
    public class KakaoUserFactory extends UserFactory {
        @Override
        protected User createUser() {
            return new KakaoUser();
        }
    }
    ```

    - 장점 : 수정에는 닫혀 있고, 확장에는 열려 있는 OCP 원칙
    - 단점 : 간단한 기능을 사용할 때보다, 더 많은 클래스를 정의해야 하는 코드량

### 참고자료
- [Factory 패턴 (2/3) - Factory Method (팩토리 메서드) 패턴](https://bcp0109.tistory.com/367)