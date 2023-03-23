## 87. 커스텀 직렬화 형태를 고려해보자

> 먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하자
>
- 개발 일정에 쫒기는 상황에서는 API 설계에 노력을 집중하는 편이 더 좋음
    - `Serializable` 을 구현하면 다음 릴리스 때 직렬화 형태를 버릴 수 없게 됨
- 유연성, 성능, 정확성 측면 고려하여 구현
- 하지만 이상적인 직렬화 형태는 **물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.**

```java
// 기본 직렬화 형태에 적합한 후보
public class Name implements Serializable {
    
    /**
     * 성. null이 아니어야함
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;
    
    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;
		
		// 나머지 코드 생략
}
```

- 논리적으로 이름, 성, 중간이름 3개의 문자열로 구성
    - 모두 `private` 임에도 문서화 주석이 달려있음
- 직렬화 형태에 포함되는 공개 API에 속하기 때문에 문서화 필요
- 기본 직렬호 형태가 적합해도, 불변식 보장과 보안을 위해 `readObject` 메서드를 제공해야 하는 경우 있음
    - `lastName` 과 `firstName` 의 경우 `NULL` 이 아님을 `readObject` 를 통해 보장

### 기본 직렬화 선택에 적합하지 않은 경우

```java
// 기본 직렬화 형태에 적합하지 않은 클래스
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    
    ... // 나머지 코드는 생략
}
```

- 논리 - 문자열을 표현
- 물리적 - 문자열들을 이중 연결 리스트로 표현
    - 직렬화를 하게 되면, 양방향 연결 정보 포함하게 된다

### 물리적 표현과 논리적 표현의 차이가 클 때 생기는 무넺

1. 공개 API가 현재의 내부 표현 방식에 종속
    - 향후 버전에서 연결 리스트를 사용하지 않더라도, 관련 처리는 필요
2. 너무 많은 공간을 차지할 수 있다
    - 위와 같이, 기본 직렬화를 사용하면 노드의 연결 정보까지 모두 포함
    - 내부 구현에 해당하니 직렬화 형태에 가치가 없고, 네트워크 전송하는 속도를 느리게 함
3. 시간이 많이 걸린다
    - 직렬화 로직은 객체 그래프의 위상에 관한 정보를 알 수 없음
    - 모든 것을 순회하며 알아보는 시간이 필요
4. 스택 오버플로를 발생
    - 기본 직렬화 형태는 객체 그래프를 재귀 순회

### 합리적인 직렬화 형태

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 지정한 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    ... // 나머지 코드는 생략
}
```

- `transient` : 이 필드는 직렬화하지 않겠다
    - 모든 필드가 `transient` 라도, `default[Write,Read]Object` 를 호출
    - 향후 릴리즈에서 `transient` 아닌 필드가 추가되는 것을 고려
- 구버전 `readObject` 에서 `defaultReadObject` 를 호출하지 않는다면, 역직렬화할 때 `StreamCorruptedException` 발생
- 메모리 절반 정도의 공간을 차지하도록 개선, 속도 측면에서도 개선

### defaultWriteObject

- `defaultWriteObject` 메서드를 호출하면, `transient` 로 선언하지 않은 모든 인스턴스 필드가 직렬화
- 따라서, `transient` 로 선언해도 되는 인스턴스 필드에 모두 `transient` 붙여주기
    - 논리적 상태와 무관한 필드라고 확신한다면 붙여주기
- 기본 직렬화를 사용하면, `transient` 필드는 기본값으로 초기화
    - 객체 - null
    - 숫자 - 0
    - boolean - false

### 동기화 객체 매커니즘

```java
private synchronized void writeObject(ObjectOutputStream e) throws IOException {
		e.defaultWriteObject();
}
```

- 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용
- `writeObject` 에서 동기화하고 싶다면, 클래스 다른 부분에서 사용하는 락 순서를 동일하게 적용

### SerialVersionUID

```java
private static final long serialVersionUID = ...
```

- 어떤 직렬화 형태를 택하든 모두 직렬버젼 UID 명시
    - 명시되어 있지 않으면, `runtime` 에 계산하여 할당하기에 성능 저하
    - 새로운 클래스라면, 어떤 `long` 을 선언해도, 중복되어도 상관 없음
- 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고, 직렬 버전 UID 를 절대 수정 X