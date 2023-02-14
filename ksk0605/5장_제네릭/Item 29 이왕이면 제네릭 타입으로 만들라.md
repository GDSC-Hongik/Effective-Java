# Item 29 : 이왕이면 제네릭 타입으로 만들라

제네릭 타입과 메서드는 사용하기 쉬운 편이지만 새롭게 제네릭 타입을 만드는 것은 생각보다 어렵다. 

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```

위 같은 경우가 제네릭이 필요한 경우인데 꺼낼 때 마다 형변환을 한다면 item 28과 같이 런타임 타입 변환 오류를 만날 가능성이 생기기 때문이다. 

## 제네릭으로 가는 길

### 첫 번째 방법

우선 제네릭 타입으로 변환해보자

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null;
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size);
    }
}
```

**→ 컴파일 오류!** 

elements는 제네릭 타입, 즉 실체화 불가 타입이고 런타임 타입안전을 지원하는 배열을 만들 수 없다. 

### 우회하기

```java
/**
* 배열 elements는 push(E)로 넘어온 E 인스턴스만 받는다.
* 따라서 타입 안전성을 보장하지만,
* 이 배열의 런타임 타입은 E[]가 아닌, Object[]다.
*/
@SuppressWarnings("unchecked")
public Stack() {
  elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

위 처럼 직접 만들지 말고 형변환 처리를 하면 되긴 되지만 타입 안전 보장 못한다는 경고가 나온다.

위와 같은 스택의 element 변수는 private이고 함수에 의해 반환되거나 다른 메서드에 인자로 전달되거나 하지 않기에 적어도 저 Stack에서는 타입 안전을 보장할 수 있다 이럴때는 경고문구를 없애는 어노테이션을 사용하여 안보이게끔하고 문서 작성을 해놓는 것이 첫번째 방법

### 두 번째 방법

elements를 처음부터 Object 배열로 만들자

```java
private Object[] elements;
```

그 다음 배열이 반환한 원소를 E로 형변환 하면

```java
E result = (E) elements[--size];
```

어김없이 Unchecked cast 경고가 뜬다. 

그러나 **push에서는 E만 허용**하고 있으므로 pop은 언제나 E만을 반환하게 되니 아까처럼 경고문구를 지워줘도 안전하다.

```java
public E pop() {
    if (size == 0)
        throw new EmptyStackException();
    
    // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
    @SuppressWarnings("unchecked")
    E result = (E) elements[--size];
    elements[size] = null;
    return result;
}
```

### 핵심 정리

> 일반화 프로그래밍을 지원하는 구현 상세는 언어마다 다르다. 자바는 제네릭이고, C++은 템플릿이다. 문법적으로 상당히 흡사한 듯 보이지만, 내부 구현은 완전히 다르다. C++은 코드를 새로 생성한다. 반면 자바는 타입 소거로 제네릭을 지원한다.
> 
> 
> **하위 호환성을 보장하려는 자바의 고육지책**인 셈인데, 이로 인해 실체화 불가 타입을 다루는 과정에서 컴파일러의 경고를 우회하는 상황이 발생한다.
> 
> 그럼에도 불구하고 제네릭이 선사하는 타입 안전성은 놓치기 아깝다. 따라서 우리는 제네릭을 적극적으로 사용함과 동시에, 자바의 언어적 특성을 바탕으로 발생할 수 있는 문제를 적절히 처리하는 방법을 익혀야 할 것이다.
> 
> 출처 : [https://gist.github.com/kth496/5d68a589bfb1b4ced135a5c7061ff69e](https://gist.github.com/kth496/5d68a589bfb1b4ced135a5c7061ff69e)
> 

***클라이언트의 직접 형변환보다 안전하고 쓰기 편한 제네릭을 사용하자!***