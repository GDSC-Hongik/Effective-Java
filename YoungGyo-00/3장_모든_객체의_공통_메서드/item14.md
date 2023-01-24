## 14. Comparable을 구현할지 고려하라

### 핵심 정리

```markdown
- 순서를 고려하는 값 클래스를 작성한다면, 꼭 Comparable interface를 작성하여 정렬, 비교, 검색 기능
  을 제공하는 Collection과 어우러지도록 개발

- compareTo 메서드에서 값을 비교할 때, "<" ">" 연산자는 쓰지 말아야 한다.

- 대신, 박싱된 기본 타입 클래스가 제공하는 static compare 메서드나 Compartor interface가 제공하
  는 비교자 생성 메서드를 사용
```

---

## Interface Comparable<T>, int CompareTo(T t)

- 3장에서 다룬 다른 메서드들과 달리, `Object` 의 메서드가 아니다
- 성격 2가지를 제외하면, `Object.equals()` 와 같다
  1. `comparTo` 는 **단순 동치성 비교, 순서**까지 비교 가능
  2. **제네릭하다**

> **Comparable을 구현**했다는 것은 인스턴스들에 `Natural Order` 가 있음을 의미한다

- 구현한 객체들의 **배열은 손쉽게 정렬**할 수 있다.

```java
// 배열 정렬 예시
Arrays.sort(a);
```

> 검색, 극단값 계산, 자동 정렬되는 `Collection` 관리도 쉽게 가능하다

```java
// Treeset 활용 예) 알파벳순 출력
public class WordList {
	public static void main(String[] args)) {
		Set<String> s = new TreeSet<>();
		Collections.add(s, args);
		System.out.println(s);
	}
}
```

> 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면, 반드시 Comparable<T> 를 구현하자

```java
public interface Comparable<T> {
	int compareTo(T t);
}
```

---

## compareTo(T t) 규약

- this 객체와 주어진 **객체의 순서를 비교**한다
  - this 객체가 주어진 객체보다 **작으면**, **음의 정수**
  - this 객체가 주어진 객체보다 **크면,** **양의 정수**
  - **같으면, 0**
  - 비교할 수 없는 타입, `ClassCastException`

> CompareTo 메서드는 다음 4**가지 규약**을 충족해야 한다

### 1. 대칭성 보장

- `sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`
- `x.compareTo(y)` 가 예외를 던질 때, `y.compareTo(x)` 도 예외를 던져야 함

### 2. 추이성 보장

- `x.compareTo(y) > 0 && y.compare(z) > 0` 이면, `x.compareTo(z) > 0`

### 3. 반사성 보장 (equals 규약과 동일)

- `x.compareTo(y) == 0` 이면, `y.compareT(x) == 0`

### 4. 동치성 결과가 equals와 같아야 한다 (권고)

- `x.compareTo(y) == 0` 이면, `x.equals(y)` 이다.
- 이를 잘 지키면, `compareTo` 로 줄지은 순서와 `equals` 의 결과가 일관
- 정렬된 컬렉션들은 동치성을 비교할 때, `equals` 대신 `compareTo` 사용

```java
BigDecimal oneZero = new BigDecimal("1.0");
BigDecimal oneZeroZero = new BigDecimal("1.00");

// Tree, TreeMap
oneZero.compareTo(oneZeroZero);

// 순서가 없는 Collection에서 사용
oneZero.equals(oneZeroZero);
```

---

## Interface Comparable vs Interface Comparator

- 두 인터페이스 모두 본질적으로 비교한다는 것 자체는 같지만, 비교 대상이 다르다

### Comparable - compareTo(T o)

- 메서드는 파라미터(매개변수)가 1개
- “자기 자신과 매개변수 객체를 비교”하는 것을 의미

```java
class Student implements Comparable<Student> {
	int age;

	Student(int age) {
		this.age = age;
	}

	@Override
	public int compareTo(Student o) {
		if (this.age > o.age) {
			return 1;
		}
		else if (this.age == o.age) {
			return 0;
		}
		else {
			return -1;
		}
	}
}
```

### Comparator - compare(T o1, T o2)

- 메서드의 파라미터(매개변수)가 2개
- “두 매개변수 객체를 비교”하는 것을 의미

```java
class Student implements Comparator<Student> {
	@Override
	public int compare(Student s1, Student s2) {
		if (s1.age > s2.age) {
			return 1;
		}
		... 이하 생략
	}
}
```

---

## CompareTo 작성 요령

- `Compareable` 타입을 인수로 받는 제네릭 인터페이스
  - 인수 타입은 **컴파일타임**에 정해진다
  - 인수 타입을 확인하거나, 형변환할 필요가 없다
  - 인수의 타입이 잘못됐다면, 컴파일 자체가 되지 않는다
  - `Null` 을 넣어 호출하면, `NullPointerException`
- 각 필드가 동치인지를 비교하는 게 아니라, 그 순서를 비교한다
  - 재귀적으로 `compareTo` 메서드를 호출한다
- Compareable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면, Comparator 를 대신 사용
- 관계 연산자 `<` , `>` 는 거추장스럽고, 오류를 유발하여 `compareTo` 메서드에서 추천하지 않음

> 아이템 10에서 구현한 CaseInsensitiveString 용 compareTo 메서드 구현

```java
// 객체 참조 필드가 하나뿐인 Comparator
public final class CaseInsensitiveString
			implements Comparable<CaseInsensitiveString> {
	public int compareTo(CaseInsensitiveString cis) {
		return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
	}
}
```

> 핵심 필드가 여러 개라면, 어느 것을 먼저 비교하느냐가 중요

- 핵심 필드부터 비교, 비교 결과가 0이 아니라면 순서가 결정되며 거기서 비교 종료
- 다음과 같이 구현하면, 간결하지만 약간의 성능 저하 유발

```java
// 기본 타입 필드가 여럿일 때의 Comparator
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드
  if (result == 0) {
		result = Short.compare(prefix, pn.prefix); // 두 번째로 중요한 필드
		if (result == 0)
			result = Short.compare(lineNum, pn.lineNum); // 마지막
	}
}
```

### 비교자 생성 메서드(Comparator construction method)

> **정적 임포트 기능**을 이용하면, 정적 비교자 생성 메서드들을 그 이름만으로 깔끔하게 사용할 수 있다

```java
// Compartor 생성 메서드를 활용한 Compartor
private static final Comparator<PhoneNumber> COMPARATOR =
			compareingInt((PhoneNumber pn) -> pn.areaCode)
				.thenComparingInt(pn -> pn.prefix)
				.thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.compare(this, pn);
}
```

- `comparingInt` 는 객체의 참조를 int 타입 키에 매핑하는 **키 추출 함수(key extractor function)**
  - 람다(lambda) 기준으로 받으며, 전화번호의 순서를 나타내는 `Comparator<PhoneNumber>` 로 반환
  - **자바 타입 추론이 불가능하므로, 타입을 명시**
- `thenComparingInt` , 두 번째 비교자 생성 메서드
  - 원하는 만큼 연달아 호출 가능
  - 타입 추론이 가능하여, 타입을 명시하지 않아도 됨

### 정적 메서드 혹은 비교자 생성 메서드를 활용하자

> 정수 오버플로 혹은 부동 소수점 계산 방식(IEEE 754)에 따른 오류를 범할 수 있으므로, 다음과 같은 코드는 작성하면 안된다

```java
// 해시코드 값의 차를 기준으로 하는 비교자 - 추이성 위배
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Objecto1, Object o2) {
		return o1.hashCode() - o2.hashCode();
	}
};
```

> 다음 두 구현 방법을 활용하자

```java
// 정적 compare 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode());
	}
}

// 비교자 생성 메서드를 활용한 비교자
static Comparator<Object> hashCodeOrder =
				Comparator.comparingInt(o -> o.hashCode());
```
