# Item 23 : 태그 달린 클래스보다는 클래스 계층구조를 활용하라

> **태그란 해당 클래스가 어떠한 타입인지에 대한 정보를 담고있는 멤버 변수를 의미한다.**
> 

예컨데 아래와 같이 모양이라는 **클래스가 사각형과 원의 역할을 동시에 담고있는 상황**을 가정해보자

```java
public class Figure {
    enum Shape {RECTANGLE, CIRCLE} // 태그

    //태그 필드 - 현재 모양을 나타낸다.
    private Shape shape;

    // 다음 필드들은 모양이 사각형일 때만 사용.
    private double length;
    private double width;

    // 다음 필드들은 모양이 원일 때만 싸용.
    private double radius;

    //원용 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    //사각형용 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    private double area() {
        switch (shape) {
            case RECTANGLE:
                return length + width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

- 가독성, switch문, 열거타입선언 등등 뭐 이래저래 단점이 많지만 가장 큰 문제는 역시 좋은 객체지향 원칙에 따른 구현이 아니라고 할 수 있겠다.

> **단일책임원칙 (Single responsibility principle) : 클래스를 변경하는 이유는 단 한 가지여야 한다.**
> 

> **개방-폐쇄 원칙 (Open/closed principle) : OCP는 확장에는 열려있으나 변경에는 닫혀 있어야한다**
> 
- 한 클래스가 두 가지의 역할을 모두 담고 있기에 위 두 가지 원칙을 부셔버리게 된다.

만약 **TRIANGLE 태그**를 추가한다고 생각해보자. 

그럼 우리는 enum에 TRIANGLE 태그를 추가해야 할 뿐 아니라 아래 switch문 분기 처리, TRIANGLE용 생성자까지 만들어야하는 매우매우 비효율적인 개발을 해야한다. 

```java
Figure figure = new Figure(0.5,0.3);
```

뿐 아니라 클라이언트 입장에서도 보면 이게 삼각형인지 사각형인지 원인지 알 수 있을까? 

여러모로 좋은 코드라고는 할 수 없을 것이다. 

## 계층 구조 클래스를 사용하자

### 방법

> **1. 루트가 될 추상 클래스 정의 후 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언. 
2. 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가 
3. 하위 클래스에서 공통으로 사용하는 데이터 필드들 모두 루트 클래스로 옮김
4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의**
> 

계층 구조 클래스로 바꾼 Figure 클래스

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {this.radius = radius;}

    @Override double area() {return Math.PI * (radius * radius);}
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override double area() {return length * width;}
}
```

- 코드는 간결하고 의미도 명확하며 오류가 있다면 컴파일 도중에 알 수 있기 때문에 런타임에 오류를 체크할 필요도 없다!
- 게다가 루트 클래스인 Figure를 건드릴 필요도 없기 때문에 아까와 같은 일도 벌어지지 않을 것.
    - 여기에 삼각형을 추가하면 루트 클래스를 상속받아서 삼각형대로 구현해주면 될 것이고
    - 정사각형을 추가하고 싶으면 사각형을 상속받아서 구현해주면 될 일이니까

> **태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각하라. 기존 클래스가 태그 필드를 사용하고 있다면 계층 구조로 리팩터링 하는 것을 고민해봅시다.**
>