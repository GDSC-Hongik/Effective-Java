# ordinal 메서드 대신 인스턴스 필드를 사용하라

ordinal 메서드는 해당 상수가 열거 타입에서 몇 번째 위치인지를 반환하는 메서드이다.

만약 열거 타입 상수와 연결된 정숫값이 필요하여 ordinal 을 사용한다면, 
추후 상수 선언 순서가 바뀌거나, 중간에 위치한 상수가 추가, 삭제될 경우에
ordinal 값에 따라 구현한 로직이 오작동할 가능성이 매우 높다.

따라서 열거 타입 상수에 연결된 값이 필요할 경우에는 인스턴스 필드를 사용하여 저장해야 한다.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), ...
    
    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```