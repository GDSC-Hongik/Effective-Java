# Item 6 : 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을때가 많다. 재사용은 빠르고 세련되다. 특히 불변 객체(아이템 17)는 언제든 재사용할 수 있다.
- 생성자 대신 정적 팩터리 메서드(item01)를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있다.
- 생성 비용이 아주 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하길 권한다.

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Patterns.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]}L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

- 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

```java
private static long sum() {
    Long sum = 0L;
    for(long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i; // sum = Long.valueOf(sum.longValue() + i);
    return sum;
}

```
