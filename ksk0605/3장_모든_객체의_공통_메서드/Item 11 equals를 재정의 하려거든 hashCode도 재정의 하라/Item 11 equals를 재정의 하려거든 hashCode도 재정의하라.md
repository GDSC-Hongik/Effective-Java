# Item 11 : equals를 재정의 하려거든 hashCode도 재정의 하라

> **equals를 재정의한 클래스 모두에서 hashCode도 재정의 해야한다.**
> 
- HashCode일반 규약을 어기게 되어 인스턴스를 HashMap이나 HashSet 같은 컬랙션의 원소로 사용할 때 문제를 일으키기 때문

### hashCode 일반규약

> **1. equals 비교에 사용되는 정보가 변경되지 않았다면 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.  단, 어플리케이션을 다시 실행한다면 달라져도 상관없다. |

2. equals(Object)가 두 객체를 같다고 판단했다면, 두 객체 hashCode는 똑같은 값을 반환해야 한다. 

3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른  객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.**
> 

2번 조항에 의해 논리적으로 같은 객체는 같은 hashCode를 반환해야 하는데 논리 적으로 같다고 하더라도 hashMap상에서는 서로다른 실객체의 hash값의 차이로 null을 반환할 수 있음 

- **예시**

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5039), "제니");

System.out.println(m.get(new PhoneNumber(707, 867, 5039)));
```

결과

![스크린샷 2023-01-24 오전 3.00.39.png](../Item%2011%20equals%EB%A5%BC%20%EC%9E%AC%EC%A0%95%EC%9D%98%20%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0%20hashCode%EB%8F%84%20%EC%9E%AC%EC%A0%95%EC%9D%98%20%ED%95%98%EB%9D%BC/Item%2011%20equals%EB%A5%BC%20%EC%9E%AC%EC%A0%95%EC%9D%98%20%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0%20hashCode%EB%8F%84%20%EC%9E%AC%E1%84%8C%20da932d91e7e5461ebd7a866719a8f298/%EA%B2%B0%EA%B3%BC.png)

## 좋은 hashCode 작성 요령

1. int 변수 result 선언 후 값 c로 초기화. 이때 c는 해당 객체의 첫번재 핵심 필드를 단계 2.a 방식으로 계산한 해시코드이다. (핵심필드란 equals 비교에 사용되는 필드) 
2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행 
    1. 해당 필드의 해시코드 c를 계산
        1. 기본 타입 필드라면, Type.hashCode(f)를 수행. Type은 해당 기본 타입의 박싱 클래스 
        2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출. 계선이 더 복잡해질 것 같으면, 이 필드의 표준형을 만들어 그 표준형의 hashCode를 호출한다. 필드의 값이 null이면 0을 사용한다(전통적으로) 
        3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(대체로 0)을 사용한다. 모든 원소가 핵심원소라면 Arrays.hashCode사용 
    2. 단계 2.a에서 계산한 해시코드 c로 result를 갱신. 
3. result 반환

- 클래스가 불변이고 해시코드 비용이 크다면 캐싱을 고려하자.