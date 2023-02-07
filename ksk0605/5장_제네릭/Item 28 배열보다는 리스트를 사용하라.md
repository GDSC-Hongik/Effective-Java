# Item 28 : 배열보다는 리스트를 사용하라

## 배열과 제네릭

- 배열은 공변 타입! 쉽게 말해 Super class에서 변하면 sub 클래스에서도 함께 변한다는 뜻.
- 이와 다르게 제네릭은 불공변. List<Super> List<Sub>는 아무상관 없다는 것.
- 이는 실체화와도 관련이 있다. 쉽게 말하면 런타임에도 타입을 검사하는가의 차이
    - 배열은 검사를 하는 실체화 가능 클래스
    - 제네릭은 안하는 실체화 불가능 클래스

```java
Object[] objectArray = new Long[1];
// ArrayStoreException 발생
objectArray[0] = "런타임에 타입 다른데?";

// 아예 컴파일 오류
List<Object> objectList = new ArrayList<Long>();
objectList.add("야 이게 컴파일이 되겠냐?");
```

> 두 가지의 근본적 차이로 인해 조화가 잘 안된다는 단점이 있다.
> 

## 왜? 답은 타입 세이프티

- 이렇게 만든 근본적인 이유는 **제네릭은 타입 안전을 보장**하겠다는 취지를 지니고 만들어졌기 때문!

```java
List<String>[] stringLists = new List<String>[1]; // (1) 
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5)
```

(1)부터 작동은 안되지만 된다고 가정하면? 

- List<String>[]을 Object[]에 넣는 것도 문제 없고
- List<Integer>를 Object[]에 넣는 것도 문제가 없으니
    - 제네릭은 소거방식, 런타임에는 타입을 지워서 확인 안해.
    - List<Integer> → List, List<Integer>[] 는 List[]
- 여기서 List<Integer>인스턴스를 저장되어있는 List<String>은 꺼내면서 자동적으로 String으로 형변환을 하게 되는데? 이때 런타임에서 ClassCastException 발생

> 사실 그래서 결국 자바는 (1)에서 컴파일 자체가 안되도록 한다.
> 

## 배열을 제네릭으로 만들려면?

넣은 것 중에 랜덤으로 하나를 꺼내게 하는 클래스 Chooser를 정의

```java
public class Chooser {
    private final Object[] choiceArray;
    
    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }
    
    // 이 메서드를 사용하는 곳에서는 매번 형변환이 필요하다.
    // 형변환 오류의 가능성이 있다.
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

아무거나 넣을 수 있는 장점이 있으나 꺼낼때마다 원하는 타입으로 형변환 해줘야 해

```java
Chooser c = new Chooser(choices);
String s = String.valueOf(c.choose());
```

- 안의 변수가 String임을 어떻게 보장할건데?

### 첫번째 시도

- 다음과 같이 고치더라도 **incompatible types** 오류가 발생한다.

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        // 오류 발생 incompatible types: java.lang.Object[] cannot be converted to T[]
        this.choiceArray = choices.toArray();
    }

    // choose 메서드는 그대로.
}
```

아래와 같이 수정하면? 

```java
this.choiceArray = (T[]) choices.toArray();
```

실행은 가능하나 **Unchecked Cast**경고가 발생!

> ***제네릭타입은 런타임에 소거된다는 사실을 잊지말자***
> 

### 두번째 시도

이 경고가 싫다면? 아래와 같이 리스트를 사용해보자

```java
class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

코드가 좀 늘고 더 느리겠지만 런타임에 ClassCastException을 만날 일은 절대 없다! 

> ***배열과 제네릭은 비슷해보이지만 공변과 불공변, 타입정보 소거 여부 등 매우 다른 타입 규칙을 가지고 있다. 배열은 런타임 중 타입 안전이지만 컴파일에는 그렇지 않고 제네릭은 그 반대이다. 그래서 섞어쓰기 쉽지 않으니 만약 같이 써야 할때 경고를 만나면 배열을 리스트로 대체하는 방법을 적용해보자.***
>