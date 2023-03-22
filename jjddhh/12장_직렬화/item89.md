# 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

객체를 싱글톤으로 유지하기 위해 아래와 같은 형식을 사용하곤 한다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
}
```

하지만 이 클래스에도 implements Serializable을 추가하는 순간 더 이상 싱글톤을 유지할 수 없게 된다.

이런 문제를 readResolve 를 사용해서 해결해보려 할 수 있는데,

```java
private Object readResolve() {
    return INSTANCE;    
}
```

이 조차도 객체 참조 타입 인스턴스 필드를 모두 transient로 선언하지 않으면 공격의 여지가 생긴다.

싱글톤이 transient가 아닌 참조 필드를 가지고 있다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화 되는데, 
이 부분에서 문제가 발생한다.

따라서 해당 문제를 해결하기 위해 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하자.

이렇게 구현한 클래스는 선언한 상수 외의 다른 객체는 존재하지 않음을 자바가 보장해준다.

