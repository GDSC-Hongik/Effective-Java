## 88. ReadObject 메서드는 방어적으로 작성하라

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period {
    private final Date start;
    private final Date end;
    
    /**
     * @param start 시작 시각
     * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }
    
    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + "-" + end; }
}
```

- 직렬화하기로 결정했다면?
    - 현재, 논리적 표현과 물리적 표현이 부합하므로 기본 직렬화 형태 사용 가능
    - `implement Serializable` 을 추가
    - 이렇게는 클래스의 주요한 불변식을 더는 보장하지 못하게 됨
- `readObject` 가 실질적으로 또 다른 Public 생성자
    - 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 정상적인 생성자로도 만들어낼 수 업슨 객체를 생성해낼 수 있음
        - 인수가 유효한지 검사
            - 검사에 실패하면, `InvalidObjectException` 을 던지며 역직렬화를 막음

            ```java
            private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
                s.defaultReadObject();
            
                // 불변식을 만족하는지 검사한다.
                if(start.compareTo(end) > 0) {
                    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
                }
            }
            ```

        - 필요하다면, 매개변수를 방어적으로 복사해야 한다

            ```java
            private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
                s.defaultReadObject();
            
                // 가변 요소들을 방어적으로 복사한다.
                start = new Date(start.getTime());
                end = new Date(end.getTime());
            
                // 불변식을 만족하는지 검사한다.
                if(start.compareTo(end) > 0) {
                    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
                }
            }
            ```



### 방어적으로 복사하지 않은 경우 발생할 수 있는 문제점

```java
public class MutablePeriod {
    //Period 인스턴스
    public final Period period;

    //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectArrayOutputStream out = new ObjectArrayOutputStream(bos);

            //유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));

            /**
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 start 필드 참조 추가
            ref[4] = 4; //참조 #4
            bos.write(ref); // 종료(end) 필드 참조 추가

            // Period 역직렬화 후 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}

// 사용 코드
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    //시간 되돌리기
    pEnd.setYear(78);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978

    //60년대로 회귀
    pEnd.setYear(60);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
}
```

- 정상 인스턴스에서 시작된 바이트 스트림 끝에 `private Date` 의 필드로의 참조를 추가하면, 가변 인스턴스를 만들어내는 문제점
    - 즉, `ObjectInputStream` 에서 인스턴스를 읽은 후 악의적인 객체 참조를 읽어 객체의 내부 정보를 알 수 있음
- Period 인스턴스는 불변식 유지한 채 생성됐지만, 의도적으로 내부의 값을 변경할 수 있음
    - 객체를 역직렬화할 때 클라이언트가 소유해서는 안되며, 객체 참조를 갖는 필드는 모두 방어적으로 복사

### 기본 readObject 를 써도 좋을지 판단하는 방법

- `transient` 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
    - 아니오라면, 커스텀 readObject를 만들어 모든 유효성 검사와 방어적 복사를 수행