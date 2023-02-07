# Item 33 : 타입 안전 이종 컨테이너를 고려하라

## 핵심정리

- 타입 안전 이종 컨테이너 : 한 타입의 객체만 담을 수 있는 컨테이너가 아니라 여러 다른 타입(이종)을 담을 수 있는
  타입 안전한 컨테이너.
- 타입 토큰 : String.class 또는 Class<String>
- 타입 안전 이종 컨테이너 구현 방법 : 컨테이너가 아니라 "키"를 매개변수화 하라!

### 타입 토큰을 사용한 타입 안전 이종 컨테이너

```java
public class Favorites {

    private Map<Class<?>, Object> map = new HashMap<>();
    // Class<?> 자체가 타입 토큰이됨
    // String.class -> 타입 토큰

    public <T> void put(Class<T> clazz, T value) {
        // clazz.cast(value) --- (1)
        this.map.put(Objects.requireNonNull(clazz), clazz.cast(value));
    }

    public <T> T get(Class<T> clazz) {
        return clazz.cast(this.map.get(clazz));
    }

    public static void main(String[] args) {
        Favorites favorites = new Favorites();
        favorites.put(String.class, "keesun");
        favorites.put(Integer.class, 2);


        // 컴파일 에러가 아니라 런타임 에러가 발생하는 방식.
        // (1) 번의 방식으로 인해 좀더 안전할 수 있지만
        // 결국 컴파일 타임에 클래스 를 로타입으로 넣기 시작하면 체킹을 못함.
        favorites.put((Class)String.class, 1); // 로 타입 변환시 강제적으로 깨트릴 수 있음

        // 이런경우엔 결국 타입이
        // List 타입으로 넘어감
       favorites.put(List<Integer>.class, List.of(1, 2, 3));
       favorites.put(List<String>.class, List.of("a", "b", "c"));

        // 결국 마지막 두번째에 넣은게 나온다.
       List list = favorites.get(List.class);
       list.forEach(System.out::println);
    }

}
```

리스트 의 제네릭을 가져올수 있는 방법이 없음..
왜냐면 T는 컴파일 타임용이고
clazz 의 타입을 꺼내봐도 리스트 타입으로 넘겨옴.
그이유는 제네릭은 런타임에 오브젝트니깐

### 수퍼 타입 토큰

익명 클래스와 제네릭 클래스 상속을 사용한 타입 토큰

상속을 사용한 경우 제네릭 타입을 알아낼 수 있다.
이경우 제네릭 타입이 제거되지 않기 때문에.

```java
public class GenericTypeInfer {

    static class Super<T> {
        T value;
    }

    static class Sub extends Super<String> {}

    public static void main(String[] args) throws NoSuchFieldException {
        Super<String> stringSuper = new Super<>(); // 사용될때에캐스팅 되는거임.
        System.out.println(stringSuper.getClass().getDeclaredField("value").getType());
        // 오브젝트가 나온다 컴파일될때는 오브젝트로 나오고
        // 사용될때에 캐스팅을 해주는거다.


        // 상속받은 클래스
        Sub sub = new Sub();
        Type type = sub.getClass().getGenericSuperclass();

        // 익명 클래스 방식
        Type type = (new Super<String>(){}).getClass().getGenericSuperclass();
        ParameterizedType pType = (ParameterizedType) type;
        // 타입 알규먼츠 배열을 가져올 수 있다 ( 제네릭 여러개 가능 하니깐. )
        Type actualTypeArgument = pType.getActualTypeArguments()[0];
        System.out.println(actualTypeArgument);
        // 스트링이 나온다!
    }
}

```

### typeRef

```java
public abstract class TypeRef<T> {
    private final Type type;
    protected TypeRef() {
        ParameterizedType superclass = (ParameterizedType) getClass().getGenericSuperclass();
        type = superclass.getActualTypeArguments()[0];
    }
    //equals hashcode
}
```

```java
// v2
    private final Map<TypeRef<?>, Object> favorites = new HashMap<>();
    public <T> void put(TypeRef<T> typeRef, T thing) {
        favorites.put(typeRef, thing);
    }
    @SuppressWarnings("unchecked")
    public <T> T get(TypeRef<T> typeRref) {
        return (T)(favorites.get(typeRref));
    }
```

### 한정적 타입 토큰

- 한정적 타입 토큰을 사용한다면, 이종 컨테이너에 사용할 수 있는 타입을 제한 할 수 있다.
- asSubclass 메서드
  메서드를 호출하는 class 인스턴스를 인수로 명시한 클래스로 형변환을 한다.

```java
public class PrintAnnotation {

    static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
        Class<?> annotationType = null; // 비한정적 타입 토큰
        try {
            // 클래스 이름을 통해서 해당 어노테이션 타입을 가져옴
            annotationType = Class.forName(annotationTypeName);
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        //getAnnotation 의 요구사항이 <? extends Annotation> 라서 바꿔줘야함.
        // 한정적인 와일드카드로 바꿈!
        Class<? extends Annotation> aClass = annotationType.asSubclass(Annotation.class)
        return element.getAnnotation(aClass);
    }

    // 명시한 클래스의 명시한 애너테이션을 출력하는 테스트 프로그램
    public static void main(String[] args) throws Exception {
        // 어노테이션을 클래스로부터 가져올려고 한다.
        System.out.println(getAnnotation(MyService.class, FindMe.class.getName()));
    }
}

```
