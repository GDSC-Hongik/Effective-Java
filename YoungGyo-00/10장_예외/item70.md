## 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

> 복구할 수 있는 상황이면 검사 예외, 프로그래밍 오류 또는 확실하지 않다면 비검사 예외

- 검사 예외 (Checked Exception)
- 런타임 예외 (Runtime Exception)
- 에러 (Error)

### CheckedException(검사 예외)

> 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용

- 호출자가 `catch` 로 잡아 처리하거나 더 바깥으로 전파하도록 강제
- 메서드를 호출했을 때, 발생할 수 있는 유력한 결과임을 전달

    ```java
    // 메서드 선언에 포함된 검사 예외
    public void copy() throws IOException {
    }
    ```


### UnChecked Exception(비검사 예외)

> 복구가 불가능할 때, 더 이상 실행해봐야 득보다는 실이 많다고 판단

- 런타임 예외 (Runtime Exception)
- 에러 (Error)

### 런타임 예외

> 프로그래밍 오류를 나타낼 때, 런타임 예외를 사용

- 런타임 예외는 대부분 전제조건을 만족하지 못했을 경우에 발생
- `ArrayIndexOutOfBoundsException` : 배열의 전제 조건이 만족하지 않았을 경우
    - 배열의 Index 는 0~index-1
- 복구할 수 있는 상황인지 프로그래밍 오류인지 명확히 구분하기 어렵다

### 에러

- JVM 자원 부족, 불변식 깨짐 등으로 더 이상 수행할 수 없는 상황
    - 자원이 일시적으로 부족한지, 수요가 순간적으로 몰렸는지, 말도 안 되는 크기의 배열이 선언된 문제인지
    - 확신하기 어려운 상황이라면, 비검사 예외를 선택하는 편
- Error 클래스를 상속해 하위 클래스를 만드는 일은 자제
- 구현하는 모든 비검사 `throwable` 은 모두 `RuntimeException` 의 하위 클래스
    - Error 를 상속하거나, 직접 `throw` 하는 일은 없도록 개발

### 예외클래스를 상속하지 않는 throwable

- 좋은 게 하나도 없으니 사용하지 말자

[자바 예외 구분: Checked Exception, Unchecked Exception](https://madplay.github.io/post/java-checked-unchecked-exceptions)