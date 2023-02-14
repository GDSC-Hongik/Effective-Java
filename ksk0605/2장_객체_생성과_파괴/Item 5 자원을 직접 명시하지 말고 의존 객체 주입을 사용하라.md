# Item 5 : 자원을 직접 명시하지 말고 의존 객체 주입을 상용하라

```java
public class MemberService {
    private final Repository repo = new MemoryRepository();
    //private final Repository repo = new JdbcRepository();

    public void join () { 
        // 가입로직...
    }
}
```

- 클래스 내부에서 직접 자원을 명시하는 것은 좋지 않다
    - 유연하지 못하고 테스트도 어렵다
    - 내가 Repository를 테스트용으로 MemoryRepository를 사용할 것인지, 실제 배포용으로 JdbcRepository를 사용할 것인지 결정할 때마다 서비스 객체에 와서 일일이 수정해야하는 불편함이 있다.
- 해결방법은 **외부에서 의존 객체를 생성자를 통해 주입**해주는 것!

```java
public class MemberService {
    public MemberService(Repository repo) {
        this.repo = Objects.requireNonNull(repo);
    }

    private final Repository repo;
    //private final Repository repo = new JdbcRepository();

    public void join () {
        // 가입로직...
    }
}
```

- **스프링**이 이걸 아주 잘 관리할 수 있도록 도와주는 프레임워크이다.