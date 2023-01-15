# Item 3 : private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
    - 함수와 같은 stateless 객체나 시스템 컴포넌트 등

## 필드 방식

- **생성자를 private로 설정**하여 **인스턴스 생성시 단 한번만 작동**하도록 함.

```java
public class Alice {
    private final String name;
    public static Alice INSTANCE = new Alice();

     private Alice() {
        this.name = "Alice";
    }

    public String getName() {
        return name;
    }
}
```

- 클라이언트 코드

```java
public class main {
    public static void main(String[] args) {
        Alice alice = Alice.INSTANCE;
        System.out.println("alice name = " + alice.getName());
    }
}
//    [출력]
//    alice1 name = Alice
//    alice2 name = Alice
```

- 권한이 있는 클라이언트에서 **AccesibleObject.setAccessible 로 private를 사용해서 생성자 호출**하는 경우말고는 안전.
    - 방어법은 생성자에 두번째 객체 생성되려할때 예외 던지게 하기

### 장점

- API에 해당 클래스가 싱글턴임이 분명하게 드러남
    - **public static 가 final 이니 다른 객체 참조가 불가능**

## 정적 팩터리 방식

- **인스턴스는 private**로 설정하고 **정적팩터리 메서드를 통해 인스턴스를 반환**하도록 함.

```java
public class AliceStaticFactory {
    private final String name;
    private static final AliceStaticFactory INSTANCE = new AliceStaticFactory();

    private AliceStaticFactory() {
        this.name = "Alice Static Factory";
    }

    public static AliceStaticFactory getInstance() {
        return INSTANCE;
    }

    public String getName() {
        return name;
    }
}
```

- 클라이언트 코드

```java
public class main {
    public static void main(String[] args) {
        AliceStaticFactory aliceStaticFactory1 = AliceStaticFactory.getInstance();
        AliceStaticFactory aliceStaticFactory2 = AliceStaticFactory.getInstance();
        System.out.println("aliceStaticFactory1 name = " + aliceStaticFactory1.getName());
        System.out.println("aliceStaticFactory2 name = " + aliceStaticFactory2.getName());
    }
}
//    [출력]
//    aliceStaticFactory1 name = Alice Static Factory
//    aliceStaticFactory2 name = Alice Static Factory
```

### 장점

- API를 변경하지 않고 싱글턴이 아니게 변경가능
- 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는 점.
    
    ```java
    Supplier<AliceStaticFactory> aliceStaticFactorySupplier = AliceStaticFactory::getInstance;
    ```