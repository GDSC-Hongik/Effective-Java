# Item 22 : 인터페이스는 타입을 정의하는 용도로만 사용하라

> **인터페이스(interface)는 [자바 프로그래밍 언어](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_(%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EC%96%B8%EC%96%B4))에서 [클래스](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%9E%98%EC%8A%A4_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))들이 구현해야 하는 동작을 지정하는데 사용는 [추상 자료형](https://ko.wikipedia.org/wiki/%EC%B6%94%EC%83%81_%EC%9E%90%EB%A3%8C%ED%98%95)이다.**
> 
- 한마디로 클래스들이 무엇을 할 수 있는지 그 명세를 알려주는 역할이라고 생각하면 된다.
- 구현한 클래스들을 모두 담을 수 있는 타입의 역할을 한다는 점에서
    - 가독성과 유지보수의 장점
    - 객체간 결합도를 낮춰 확장에는 열려있고 변경에는 닫혀있어야 하는 좋은 객체지향 원칙의 토대가 되는 중요한 개념이다.

**중요한 것은 인터페이스의 역할을 잘 이해하는 것!**

## 이 역할 혹 지침에 맞지 않는 구현 예

```java
public interface PhysicalConstans {
    
    // 아보가드로 수
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    
    //볼츠만 상수
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    
    // 전자 질량
    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

- static final 로만 가득찬 인터페이스를 상수 인터페이스라고 하는데 이 경우가 대표적인 잘못 사용한 예
- 이는 인터페이스의 근본 역할에서 지나쳐 그 내부 구현을 훤히 드러내는 행위와 동일
- 만약 더 이상 저 상수들이 필요하지 않아졌다고 해도 구현클래스들의 바이너리 호환성을 위해 계속 남겨둬야 한다.

> **바이너리 호환성 : 뭔가를 바꾼 이후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황.**
> 

## 다른 옵션?

상수를 공개할 목적이라면 열거 타입을 인터페이스나 클래스 내부의 정의해주는게 낫고

```java
public enum Day{ MON, TUE, WED, THU, FRI, SAT, SUN};
```

아니면 상수 유틸리티 클래스를 사용하자

```java
public class PysicalConstants{
      private PysicalConstants(){}; // 인스턴스화 방지
      public static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      public static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```

> **인터페이스는 타입을 정의하는 용도로만 사용해라. 상수 공개용 수단같은 용도에 맞지 않는 수단으로는 NO!**
>