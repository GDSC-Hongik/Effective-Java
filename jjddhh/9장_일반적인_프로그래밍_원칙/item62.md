# 다른 타입이 적절하다면 문자열 사용을 피하라

문자열(String)은 해당 값이 정말로 텍스트를 의미할 때 사용해야 한다.

문자열은 열거 타입이나 혼합 타입을 대신하기에 적합하지 않다.

```java
String compoundKey = className + "#" + i.next();
```
예를 들어 이러한 코드가 있다고 하자.

문자열에 들어있는 요소를 개별로 접근하려 하면 파싱해야하고, 해당 과정은 느리며 여러 문제가 발생할 가능성이 있다.

뿐만 아니라 compoundKey 는 새로운 기능을 추가할 수 없으며 String 이 제공하는 메서드만 사용할 수 있다.

또한 문자열은 권한을 표현하기에 적합하지 않다.