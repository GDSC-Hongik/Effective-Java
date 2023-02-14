# Item 2 : 생성자에 매개변수가 많다면 빌더를 고려하라

- 생성자나 정적 팩터리 메서드나 둘 다 매개변수가 많아지면 쉽지 않아짐
- 매개변수가 6개일 때 우리는 생성자 옵션을 최대 6개까지 생각해야 하기 때문이다.

### 점층적 생성자 패턴

```java
// 점층적 생성자 패턴
public class fufu {
    private int a;
    private int b;
    private int c;
    private int d;
    private int e;
    private int f;
    
    public fufu(int a) { 
        this(a,0);
    }
    
    public fufu(int a, int b) {
        this(a, b, 1);
    }
    
    public fufu(int a, int b, int c){
        this(a, b, c, 2);
    }
    
    ...

    public fufu(int a, int b, int c, int d, int e, int f) {
        this.a = a;
        this.b = b;
        this.c = c;
        this.d = d;
        this.e = e;
        this.f = f;
    }
}
```

- 원치 않는 매개변수도 굳이 넣어줘야하는 상황이 생길 수 있음
- **매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어려움.**

### 자바빈즈 패턴

```java
public class fufu {
    private int a = 0;
    private int b = 0;
    private int c = 0;
    private int d = 0;
    private int e = 0;
    private int f = 0;

    public fufu() {

    }

    public void setA(int a) {
        this.a = a;
    }

    public void setB(int b) {
        this.b = b;
    }

    public void setC(int c) {
        this.c = c;
    }

    public void setD(int d) {
        this.d = d;
    }

    public void setE(int e) {
        this.e = e;
    }

    public void setF(int f) {
        this.f = f;
    }
}
```

- 객체 하나만드는데에 **여러번 매서드 호출**이 일어나야 함
- 객체의 완전한 생성 전까지 **일관성이 깨짐**
    - 인스턴스가 의도한 바와 같이 생성되었는지(올바른 값이 들어있는지) 확인이 어려움
    
    ```java
    fufu.setA(10);
    // 요 사이에 무슨 일이 일어나서 b에 11이 들어갔을거라고 확신할 수 없지
    // 스레드 안정성을 위해선 추가 작업이 필요하게 함 ex.freezing (lock)
    // 동시성 문제도 있지 않을까? 
    fufu.setB(11);
    ```
    
- **불변 클래스 불가**
    - 언제 초기화 될지 모르니 final 설정이 안됨

### 빌더 패턴

- 점층적 생성자 패턴의 안전성 get
- 자바빈즈패턴의 가독성 get

```java
public class fufu {
    private final int a;
    private final int b;
    private final int c;
    private final int d;
    private final int e;
    private final int f;

    public static class Builder{
        private final int a;
        private final int b;

        private int c = 0;
        private int d = 0;
        private int e = 0;
        private int f = 0;

        public Builder(int a, int b) {
            this.a = a;
            this.b = b;
        }

        public Builder c(int c) {
            this.c = c;
            return this;
        }

        public Builder d(int d) {
            this.d = d;
            return this;
        }

        public Builder e(int e) {
            this.e = e;
            return this;
        }

        public Builder f(int f) {
            this.f = f;
            return this;
        }

        public fufu bulid() {
            return new fufu(this);
        }
    }

    private fufu(Builder builder) {
        a = builder.a;
        b = builder.b;
        c = builder.c;
        d = builder.d;
        e = builder.e;
        f = builder.f;
    }
}
```

- 사용 예시

```java
fufu f = new fufu.Builder(1,2).c(3).e(5).build();
```

- python / scalar 와 같은 언어의 명명된 선택적 매개변수를 흉내낸 것
- 계층적으로 설계된 클래스와 함께 쓰기 좋음.