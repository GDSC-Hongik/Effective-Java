# Item 1 : 생성자 대신 정적 팩터리 메서드를 고려하라.

클래스는 생성자와 별도로 정적 팩터리 메서드를 제공할 수 있다

---

### 장점

1.  이름을 가질 수 있다.

```java
public class Order {

    private boolean prime;

    private boolean urgent;

    private Product product;

    private OrderStatus orderStatus;
    // prime 처럼 이름을 가질 수 있음
    public static Order primeOrder(Product product) {
        Order order = new Order();
        order.prime = true;
        order.product = product;

        return order;
    }
    // urgent 처럼 이름을 가질 수 있음
    public static Order urgentOrder(Product product) {
        Order order = new Order();
        order.urgent = true;
        order.product = product;
        return order;
    }

    public static void main(String[] args) {

        Order order = new Order();
        if (order.orderStatus == OrderStatus.DELIVERED) {
            System.out.println("delivered");
        }
    }

 }
```

이름을 가질 수 있기때문에 명확 해짐.
boolean 값을 매번 안넘겨도 됨!

2. 호출될 때마다 인스턴트를 새로 생성하지 않아도 된다.

```java
// 이미 세팅한 타입
private static final Settings SETTINGS = new Settings();
public static Settings getInstance() {
    return SETTINGS;
}
```

싱글턴 비슷하게 만들 수 가 있음

3.  반환 타입의 하위 타입 객체를 반환 할 수 있는 능력이 있다.

```java
@Getter
@AllArgsConstructor
public class DuDoongCodeException extends RuntimeException {
    private BaseErrorCode errorCode;

    public ErrorReason getErrorReason() {
        return this.errorCode.getErrorReason();
    }
}

@Getter
@Builder
public class ErrorReason {
    private final Integer status;
    private final String code;
    private final String reason;
}

public class InvalidTokenException extends DuDoongCodeException {
    //InvalidTokenException 타입이 아닌 DuDoongCodeException 을 반환.
    public static final DuDoongCodeException EXCEPTION = new InvalidTokenException();

    private InvalidTokenException() {
        super(GlobalErrorCode.INVALID_TOKEN);
    }
}
//이런식으로 사용가능
DuDoongCodeException exception = InvalidTokenException.EXCEPTION;
ErrorReason errorReason = exception.getErrorReason();
// 저는 이렇게 받아서 스웨거 커스텀? 할때 그냥 부모 타입으로 바로받으니깐
// 부모 참조없이 타입으로 그냥 적용..
```

4.  입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.(EnumSet)

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
    // 이넘의 길이가 64보다 작으면 레귤러
        return new RegularEnumSet<>(elementType, universe);
    // 크면 점보
    else
        return new JumboEnumSet<>(elementType, universe);
}

```

5.  정적 팩터리 매서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. ( 서비스 제공자 프레임워크 , ServiceLoader)

i.구현체의 동작을 정의하는 서비스 인터페이스(service interface)
ii. 제공자가 구현체를 등록할 때 사용하는 제공자 등록 API(provider registration API)
iii. 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API(service access API)
iv. 서비스 제공자 인터페이스(service provider interface) : 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해 줌. 유연한 정적 팩터리 : 클라이언트는 서비스 접근 API를 사용할 때 원하는 구현체의 조건을 명시할 수 있으며, 조건을 명시하지 않으면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다.

---

### 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
   자바 독에 생성자 부분에 생기는게 아님.

---

### 관련 주제

1.  enum 열거타입 안전한 싱글톤 생성가능

컴파일 시점에 이미 생성되어버려서
리플렉션으로도 private 한걸 public으로 못바꾸서 접근 불가.

```java
public enum ActionEnum {
ACTION(new App())

private final App action;
// 생성자도 가능, 메서드도 가능
ActionEnum(App action) {this.action = action;}

}
```

2.  플라이 웨이트 패턴
    가볍게 중복 정보드를 제거해 나가는 패턴

3.  인터페이스에 정적 메소드 (default method , private static 메서드)

```java
public interface HelloService {

    String hello();

    static String hi() {
        prepareMessage();
        return "hi";
    }

    static private void prepareMessage() {
    }

    static String hi1() {
        prepareMessage();
        return "hi";
    }

    static String hi2() {
        prepareMessage();
        return "hi";
    }

    default String bye() {
        return "bye";
    }
}
```

4.  서비스 제공자 프레임 워크
    핵심은 사용자가 몰라도 된다? 라는 거.

```java
import java.util.ServiceLoader;
// 자바에서 기본으로 제공하는 서비스 로더.
ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);
// 클래스 타입으로 땡겨올 수 있음
Optional<HelloService> helloServiceOptional = loader.findFirst();
helloServiceOptional.ifPresent(h -> {
    System.out.println(h.hello());
});
// di 스럽게 할 수있음
```

5.  리플랙션
    클래스 로더로 부터 읽힌 정보가 jvm 메모리 영역에 쌓임.
    어노테이션, 필드 , 메서드 등 클래스 내부에 담긴 정보를 얻을 수 있음

```java
// 클래스 정보 가져오기
Class<?> aClass = args[i].getClass();
// 필드 이름
String capitalize = StringUtils.capitalize(identifier);
// private 필드를 getter로 접근.
Object result = aClass.getMethod("get" + capitalize).invoke(args[i]);

// 또는 private 접근 어세스를 true로 변경가능
field.setAccessible(true);
value = field.get(실제 오브젝트);

return String.valueOf(result);
```

6.  명명방식

#### 정적 팩토리 메서드 명명 방식

- from : 매개변수를 하나 받아서 해당 타입의 인스ㅓㄴ스를 반환하는 형변환 메서드

```
Date d = Date.from(instant);
```

- of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드

```
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```

- valueOf : from과 of이 더 자세한 버전

```
BigInteger prime : BigInteger.valueOf(Integer.MAX_VALUE);
```

- instance 혹은 getInstance : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.

```
stackWalker luke = StackWalker.getInstance(options);
```

- create 혹은 newInstance : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.

```
Object newArray = Array.newInstance(classObject, arrayLen);
```

- getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입.

```
FileStore fs = Files.getFileStore(path);
```

- newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입.

```
BufferedReader br = Files.newBufferedReader(path);
```

- type : getType과 newType의 간결한 버전

```
List<Complaint> litany = Collections.list(legacyLitany);
```
