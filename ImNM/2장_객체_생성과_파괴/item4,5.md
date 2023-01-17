# Item 4 : 인스턴스화를 막으려거든 private 생성자를 사용하라

네그럼됩니다..

```java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용)
	private UtilityClass() {
		throw new AssertionError();
	}
}
```

# Item 5 : 자원을 직접 명시하지말고 의존 객체 주입을 사용하라

의존성.. 주입하라는 이야기!
인터페이스로 다형성 제공해야 모킹도 쉬워져서 테스트하기 편하다.

```java
public class Concert {

    public void start(Supplier<Singer> singerSupplier) {
        Singer singer = singerSupplier.get();
        singer.sing();
    }

    public static void main(String[] args) {
        Concert concert = new Concert();
        // 팩터리를 서플라이어로 받을수도 있고..
        concert.start(()->Elvis.getInstance());
    }
}

public class Elvis implements Singer {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // 이 메서드는 보통 클래스 바깥(다른 클래스)에 작성해야 한다!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();

        System.out.println(Elvis.getInstance());
        System.out.println(Elvis.getInstance());
    }

    @Override
    public void sing() {
        System.out.println("my way~~~");
    }
}
```

이렇게 supplier 로 넘겨주는거 도 가능... 진짜 아름다운 쏘스네
저렇게 되면 팩토리도 소스에 올수있음?
