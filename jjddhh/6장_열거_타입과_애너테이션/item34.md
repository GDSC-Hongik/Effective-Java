# int 상수 대신 열거 타입을 사용하라

정수 열거 패턴의 경우 타입 안전을 보장하지 못하며 표현력도 좋지 않다.

열거 타입 사용을 통해 이런 단점을 해결할 수 있다.

열거 타입은 인스턴스들이 싱글턴임이 보장되며, 컴파일타임 타입 안정성을 제공한다.

이뿐만 아니라 열거 타입에는 메서드나 필드를 추가할 수 있고, 임의의 인터페이스를 구현하게 할 수도 있다.

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    
    ...
    
    private final String symbol;
    
    Operation(String symbol) { this.symbol = symbol; }
    
    public abstract double apply(double x, double y);
}

```

열거 타입 상수를 특정 데이터와 연결짓기 위해 위의 코드처럼 인스턴스 변수를 선언하고 생성자로 데이터를 받는 방식을 사용할 수 있으며,

추상 메서드를 선언해두면 상수별로 해당 메서드를 재정의 하도록 할 수도 있다.

이렇듯 열거 타입은 정수 열거 패턴을 사용했을 때는 할 수 없던 기능들을 구현할 수 있다.

### 분기로직 작성시

열거 타입을 사용하다보면 열거 타입안에 switch 문을 사용해서 값에 따른 분기처리를 하는 로직을 구현할 때가 있다.

이렇게 switch 문을 사용하여 구현하면 코드는 간결해질지 몰라도 상수의 추가, 
삭제가 발생할 가능성이 있을때는 관리 관점에서 매우 위험한 코드가 된다.

switch 문을 사용함으로 발생하는 문제를 해결하기 위한 방법으로는 새로운 상수를 추가할 때, 구현해둔 '전략'을 선택하도록 하는 것이다. <br/>
> 이를 '전략 열거 타입 패턴' 이라 한다.

이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연하다.

```java
enum PayrollDay {
    MONDAY, TUESDAY, ... ,
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND)
    
    private final PayType payType;
    
    PayrollDay(PayType payType) { this.payType = payType; }
    
    // 전략 열거 타입
    enum Paytype {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };
        
        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;
        
        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
    
}
```

하지만 상황에 따라 switch 문도 좋은 선택이 될 수 있으니, 상황에 따라 판단하여 어떤 방법을 사용할 지 선택해야 한다. 

