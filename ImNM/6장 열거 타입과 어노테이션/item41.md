# Item 41 : 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 핵심정리

- 아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스를 마커인터페이스.
- 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있다.
- 마커 인터페이스는 적용 대상을 더 정밀하게 지정할 수 있다.
- 마커 애너테이션이 마커 인터페이스보다 나은 점은 거대한 에너테이션 시스템의 지운얼 받는다.

### 1. 인스턴스를 구분하는 타입으로 쓸수있다.

serializable 인터페이스가 그 좋은 예시.
직렬화 역직렬화가 가능하다는것을 알려준다.

```java
// write object in ObjectIOutput stream.
public final void writeObject(Object obj) throws IOException {
        if (enableOverride) {
            writeObjectOverride(obj);
            return;
        }
        try {
            writeObject0(obj, false);
        } catch (IOException ex) {
            if (depth == 0) {
                writeFatalException(ex);
            }
            throw ex;
        }
    }
```

만일 Object 형식대신 Serializable 타입으로 받았다면, 컴파일 타임에 오류를 발견할 수 있을것이다.
하지만 마커 어노테이션의 경우 런타임에 해당 타켓에 어노테이션이 붙어있는지 없는지 검사를 해야하거나,
동적으로 정보를 가져와야한다. 물론 장점일 경우가 있지만 타입이라면 부적절한 예시다.

### 2. 적용대상을 더 정밀하게 적용 할 수 있다.

```java
@Target(ElementType.TYPE)
// 적용 대상은 클래스, 인터페이스 , 열거 타입 , 애너테이션
```

범위가 너무 넓다.

반면 Set의경우 collction의 마커 인터페이스로 볼수 있음.

```java
public interface Set<E> extends Collection<E>
```

메서드 몇개만 살짝 수정했을 뿐임.

### 결국 어떤 때 적절하게 사용할 수 있을까?

마커를 클래스나 인터페이스에 적용해야 한다면.
질문을 던져보자.

`이 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있을까?`

답이 그렇다면 마커 인터페이스를 써야한다.
컴파일 타임에 오류를 잡아 낼 수 있기 때문이다.

타입을 정의할 거면 인터페이스를 써라.
타입을 정의할 것이 아니라면 인터페이스를 사용하지마라.
