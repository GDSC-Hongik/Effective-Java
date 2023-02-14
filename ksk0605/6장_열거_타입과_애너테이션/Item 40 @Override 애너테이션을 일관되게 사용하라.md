# Item 40 : @Override 애너테이션을 일관되게 사용하라

`@Override` 

- 상위타입의 메서드를 재정의 했음을 명시적으로 표시하는 애너테이션
- 습관적으로 재정의 하는 함수에는 무조건 습관적으로 이 애너테이션을 붙이면 여러가지 버그들을 예방할 수 있다.

### 잘못 재정의 한 예시

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size()); // 출력 : 260
    }
}
```

aa,  bb, cc … ,zz까지 26개만 있어야 할 것 같은 `Set`의 사이즈가 어째서인지 260으로 출력된다. 

equals 메서드를 재정의하려면 hashcode 메서드도 재정의 해야한다는 사실을 잘 지켰지만 equals의 인자 조건이 잘못되었다. equals를 재정의 하려면 인자의 타입이 Object여야 하기 때문.

### 올바르게 고친 예시

```java
@Override public boolean equals(Object o) {
        if (!(o instanceof Bigram2))
            return false;
        Bigram2 b = (Bigram2) o;
        return b.first == first && b.second == second;
}
```

`@Override` 애너테이션이 재정의 하려는 함수를 잘못 작성하였을때 알아서 컴파일 오류를 내주니 무조건 작성하자!

### 예외 사항

**구체 클래스에서 상위 클래스의 메서드를 재정의**할 때는, 굳이 `@Override`를 달지 않아도 된다. 컴파일러가 자동으로 사실을 알려주기 때문!

### 핵심정리

> 재정의한 모든 메서드에 @Override 애너테이션을 달면 컴파일러가 알아서 실수를 감지해준다. 예외는 구체 클래스에서 상위클래스의 추상 메서드를 재정의했을 때인데 이것도 단다고 해서 문제가 안되니 참고하자!
>