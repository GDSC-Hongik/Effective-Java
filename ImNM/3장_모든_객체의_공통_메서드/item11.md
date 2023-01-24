# Item 11 : equals를 재정의하려거든 hashCode도 재정의하라

## HashCode 규약

- equals 비교에 사용하는 정보가 변경되지 않았다면 hashCode 는 매번 같은 값을 리턴해야하 한다 . ( 변경되거나, 애플리케이션을 다시 실행 했다면 달라질 수 있다.)

- equals() 로 두 객체를 같다고 판단했다면 두 객체의 hashCode는 똑같은 값을 반환해야한다

- equals()가 두 객체를 다르다고 판단했더라도, hashCode를 각각 다른값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다

## 왜 HashCode 도 재 정의 해야하는 지?

```java
public class PhoneNumber {
    private final int areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        PhoneNumber that = (PhoneNumber) o;
        return areaCode == that.areaCode && prefix == that.prefix && lineNum == that.lineNum;
    }
    // 해쉬 코드는 재정의 안해버림

    public static void main(String[] args) {
        Map<PhoneNumber, String> m = new HashMap<>();
        PhoneNumber phoneNumber = new PhoneNumber(707, 867, 5309);
        PhoneNumber phoneNumber2 = new PhoneNumber(707, 867, 5309);
        System.out.println(phoneNumber2.equals(phoneNumber)); // true

        m.put(phoneNumber, "제니");
        System.out.println(m.get(phoneNumber2)); // null
    }

```

### map 에서 정보를 저장하고 가져오는 방법.

해쉬코드로 key 값을 잡음.
해쉬코드가 만일 겹친다(충돌(hash collision) 이 일어난다).
linked list 또는 tree (현재는 트리구조래요) 구조로 줄줄이 저장한다.
그러면 저장한 줄줄이 목록에서 equals 비교를 통해서 가져옴.

따라서 위 방법에선 hashcode 를 재정의 하지 않았기 때문에 map 에서 겟을 못하는 것임.

### hashcode 를 그냥 같게 하면 안됨?

```java
@Override
public int hashCode() {
   return 42;
}
```

그냥 값을 내려주는방법.
동작은 함!

42라는 똑같은 값만 내주어 해시버킷하나에 노드들이 쌓이게 되어 연결리스트처럼 동작한다 =>
해시코드가 42인 객체들의 노드가 버킷에 쌓이게 되고
링크드리스트(연결 리스트)로 버킷을 관리하게되어 조회시 속도가 O(1) -> O(N)으로 느려진다

자바8부터는 연결리스트의 노드가 8개 이상이되면 그때부터 레드블랙트리로 관리합니다 O(LogN)
즉 알고리즘 자체가 그냥 느려지게 되어버림.

### 올바른 방법

```java
// 전형적인 해쉬코드 메서드
    @Override public int hashCode() {
        int result = Short.hashCode(areaCode); // 1
        result = 31 * result + Short.hashCode(prefix); // 2
        result = 31 * result + Short.hashCode(lineNum); // 3
        return result;
    }
```

왜 31 이냐? 소수, 홀수이며,
모든 글자들을 해쉬화했을때 충돌이 제일 적게 일어났던게 31이라는 연구 결과가 있다네여

그냥 롬복의 equalshashcode 쓰는게 맘편함

### 해쉬코드의 동시성? 굳이 싶긴함.

```java
// 지연 초기화
    private int hashCode;

    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Integer.hashCode(areaCode);
            result = 31 * result + Integer.hashCode(prefix); // 여러 스레드가 동시 접근 가능
            result = 31 * result + Integer.hashCode(lineNum);
            hashCode = result; // result 값이 달라 질 수 있음
        }
        return result;
    }


```

```java
// synchronized를 최소화
// 락의 범위를 최소화 하는게 진짜 진짜 중요.
// 더블 체크드 락킹 방식
private volatile int hashCode; // 자동으로 0으로 초기화된다.
// volatile 은 캐싱을 안하고 ( 더티 체킹 ) 바로 메모리에 write 하는 키워드
// 메모리 캐싱 ( LRU 같은 방식 ) 을하게 되면 쓰레드마다 다른걸 가져와 버릴 수있음
    @Override public int hashCode() {
        if (this.hashCode != 0) {
            return hashCode;
        }
        // synchronized 키워드로 클래스 락 사용.
        synchronized (this) {
            int result = hashCode;
            if (result == 0) {
                result = Short.hashCode(areaCode);
                result = 31 * result + Short.hashCode(prefix);
                result = 31 * result + Short.hashCode(lineNum);
                this.hashCode = result;
            }
            return result;
        }
    }
```

- threadLocal
- 불변 객체 사용
