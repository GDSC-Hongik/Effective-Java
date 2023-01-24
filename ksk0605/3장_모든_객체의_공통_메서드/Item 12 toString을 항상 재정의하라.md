# Item 12 : toString을 항상 재정의하라

> **toString을 잘 구현한 클래스는 사용자에겐 즐겁고 시스템에겐 디버깅하기 쉽다!**
> 

### Object의 기본 toString

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

클래스이름과 해시코드를 반환하는 해당 기본 메서드는 모든 클래스에서 유용해보이지 않는다. 따라서 방법은 **클래스에 적합하게 항상 재정의** 하는 것!

특히 컬랙션에서 유용하게 사용할 가능성이 크니 유의하는 것이 좋다!

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5039), "제니");
m.put(new PhoneNumber(123,4234,3223), "겨미");

System.out.println(m.toString());
```

이 코드에서

> {item11.PhoneNumber@48140564=겨미, item11.PhoneNumber@1d251891=제니}
> 

보다는 

> {123-423-3223=겨미, 707-867-5039=제니}
> 

가 나으니까.

이때 **포맷을 명시하거나 하지 않더라도 꼭 의도를 명시**하는 것이 좋다!

```java
/**
 * 이 전화번호의 문자열을 반환
 * 이 문자열은 "XXX-YYY-ZZZZ" 12글자로 구성된다.
 * XXX는 지역코드 YYY는 프리픽스 ZZZZ는 가입자 번호이다.
 * 각 대문자는 10진수 숫자 하나를 나타낸다.
 *
 * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면
 * 앞에서 부터 0으로 채워나간다.
 * **/
@Override
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

단 유틸리티 클래스라면 필요 굳이? 이미 Object.toString이 잘 표현해주고 있으니까.

> ***유틸리티 클래스* 

비슷한 기능의 메서드와 상수를 모아서 캡슐화 하는 것**
>