# 가변인수는 신중히 사용하라

### 가변인수란?

```java
static int sum(int... args){
    int sum = 0;
    for(int arg : args)
        sum += arg;
    return sum;
}
```

위 코드처럼 `int... args` 를 가변인수라 하고, 가변인수는 명시한 타입의 인수를 0개 이상 받을 수 있다.

가변인수의 동작 방식은 메서드 호출시 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고, 
인수들을 이 배열에 저장하여 메서드에 넘겨주는 방식이다.

만약 인수가 1개 이상이어야 할 경우에 if 예외문을 통해 만드는 경우가 있는데, 이러한 설계는 좋지 않다.

```java
static int sum(int... args){
    if(args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    
        ...
        
}
```

이 구현의 가장 큰 문제는 인수를 0개만 넣어 호출시에 컴파일타임이 아닌 런타임에 예외가 발생한다는 것이다.

따라서 이와 같은 상황에서는 아래와 같은 구현이 더 바람직하다.

```java
static int sum(int firstArg, int... remainingArgs){
    int min = firstArg;
    for(int arg : remainingArgs)
        if(arg < min)
            min = arg;
    return min;
}
```

### 성능

성능에 민감한 상황에서는 메서드 호출시마다 배열을 새로 할당하는 가변인수는 걸림돌이 될 수 있다.
이러한 성능 문제를 해결하기 위한 패턴이 있다.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

일반적인 상황에서는 이러한 패턴을 통해 얻는 큰 이득이 없지만, 특수한 상황에서는 성능적으로 도움이 될 수 있다.

> `EnumSet` 정적 팩토리도 위와 같은 방식으로 구현되어 있다