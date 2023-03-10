# Item 10. equals는 일반 규약을 재정의하라
equals는 재정의하기 쉬워 보이지만 절대 그렇지 않다. <Br>

자칫하면 끔찍한 겨로가를 초래하는 함정이 곳곳에 도사리고 있다.. <br>

**꼭 필요한 경우가 아니면 equals를 재정의하지 않는다.** <br>
많은 경우에 Object의 equals가 우리가 원하는 비교를 정확히 해준다! <br>
그래도 해야할 때가 있는데, 그 클래스들의 **핵심 필드들 모두를 빠짐없이, 다섯 가지 규약을 지켜나가며 비교해야 한다!** <br>
이제 재정의하지 않는 경우와, 재정의 하기 위한 다섯가지 규약을 살펴보자!

## 1. equals를 재정의하지 않는 경우
아래의 상황들 중 하나에 해당된다면! 재정의 하지 말라
1. **각 인스턴스가 본질적으로 고유한 상황** ex) Thread
2. **인스턴스의 논리적 동치성을 검사할 일이 아예 없는 케이스.**
3. **상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는 상황:** 재활용이 가능하다면, 굳이 재정의하지 않는다.
4. **클래스가 private이고 equals를 호출할 일이 없다.** <br> 단, 혹시 호출되는 상황을 막고 싶다면 AssertionError를 던져주면 그만이다.
```java
@Override public boolean equals(Object o) {
  throw new AssertionError();
}
```

## 2. 재정의 하는 경우
그럼 언제 재정의 해야 할까..? <br>
바로 **객체의 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 객체 식별성을 비교하고 있을 때이다.** <br>

주로 값 클래스들이 이런 상황에 처해있다. <br>
값 클래스를 사용하다가 equals를 호출하는 프로그래머는 객체가 같은지 다른지가 아니라, 가지고 있는 값이 같은지가 궁금할것이다. <br>
그래서 만약 값 클래스의 equals가 객체 식별성을 비교하고 있다면 equals를 재정의 해주자. <Br>
값의 비교가 가능해 지는것은 물론이고 **Map의 키와 Set의 원소로 사용할 수 있게 된다.** <br>
물론 값 클래스 중 값이 같은 인스턴스가 만들어지지 않음을 보장하는 경우에는 재정의 하지 않다도 된다. 그런 경우에는 결국 객체 식별성이 곧 객체의 논리적 동치성이기 때문이다. (Enum이 그렇다.) <br>

## 3. equals 재정의 할때의 규약
equals 재정의 할때 따라야 하는 규약을 확인해보자.
```
equals 메서드는 동치관게를 구현하며 다음을 만족한다.
1. 반사성: null이 아닌 모든 참조 값에 대해 x.equals(x) 는 true이다.
2. 대칭성: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y) 가 true이면, y.equals(x) 또한 true이다.
3. 추이성: null이 아닌 모든 참조 값 x, y, z 에 대해 x.equals(y) 가 true이고, y.equals(z) 또한 ture이면, x.equals(z) 또한 true이다. 
4. 일관성: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출해도 항상 같은 값을 반환한다.
5. null-아님: null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false이다
```

위의 5가지 규약을 지켜내며 재정의 해야한다. <br>
하나 하나 자세히 뜯어보자.

### 3.1 반사성
객체는 자기 자신과 같아야 한다는 뜻이다. <br>
일부러 어기지 않는다면 어기기가 더 어려운 요소

### 3.2 대칭성
두 객체는 서로에 대해 동치 여부를 같게 대답해야 한다는 것이다. <br>
둘이 같으면 서로가 서로를 같다고 말 해야하고, 다르다면 서로가 서로를 다르다고 말해야 한다! <br>
대칭성은 은근 어기기가 쉬운데, **하위 인스턴스에서 equals를 재정의하여 상위 인스턴스와 `같다`라고 답하게 했지만, 반대로 상위 인스턴스에서 하위 인스턴스에 대해 equals를 호출 했을 때 다르다고 하는 경우가 있다.** <br> 
이 경우는 하위 클래스야 상위 클래스를 알도록 잘 재정의 했지만, 상위 클래스는 하위 클래스의 존재를 모르기 때문에 발생한다. <br>

### 3.3 추이성
추이성은 x와 y가 같고, y와 z가 같다면, x와 z는 같다는 규칙이다. <br>
추이성 또한 간단하지만 자칫하면 어기기 쉬워서 조심해야 한다. <br>
이는 특히 **구체 클래스를 확장하며 새로운 필드를 추가하면서 equals를 만족시킬 방법을 찾느라 발생한다.** <Br>
이는 **객체 지향 언어의 동치관계에서 나타나는 근본적인 문제로,** 대칭성을 위반하거나 추이성을 위반할 수 밖에 없다. <br>
새로운 필드가 계속해서 추가되는데, 이를 확장되기 이전의 클래스가 인지할 수 없음은 당연하다.
<!-- 그럼, equals를 클래스 비교 메서드를 바꿔주면서 구체 클래스를 확장한다면 문제가 해결되는걸까? <br>
그렇지 않다.  -->
 
### 3.4 일관성
두 객체가 같다면, 앞으로도 여전히 같아야 한다는 듯이다. <br> 
불변 객체의 경우 신경쓰지 않아도 되겠지만, 가변 객체의 경우 매번 같은 값은 반환하도록 각별히 신경 써주어야 한다. <br>
equals의 판단에 신뢰할 수 없는 자원이 끼어드는 것을 각별히 주의해야 한다. <Br>
equals는 항시 메모리에 존재하는 객체만을 사용한 '결정적 계산'만을 수행해야 한다. 


### 3.5 null-아님
공식 이름은 아니지만, 모든 객체는 null과 다른 객체여야 한다는 뜻이다. <br>
사실 어떤 객체와 null이 같다고 판단하는 상황을 조심하란 것이 아니라, **NullPointerException을 던지는 코드를 조심하라는 규약이다.** <br>

예를 들어
```java
@Override public boolean equals(Object o) {
  if (o == null)
    return false;
}
```
위와 같은 코드는 `ClassCastException` 에러를 반환한다. 차라리 아래와 같이 코드를 짜면, 예외가 아닌 **false값을 반환한다.**
```java
@Override public boolean equals(Object o) {
  if (!(o instanceof MyType))
    return false;
  MyType myType = (MyType) o;
  ...
}
```

## 4. 양질의 equals 메서드를 구현하는 방법
이제까지의 내용을 종합해 단계별로 나타내본다.

1. **`==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.** <br> 자기 자신이면 true를 반환한다. 이는 성능 최적화로 복잡한 상황에서 힘을 발휘할 것이다. (참조 타입은 equals로, float와 double은 compare 메서드로 비교)
2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다. 아닌 경우 false를 반환한다.** 
3. **입력을 올바른 타입으로 형변환한다.** <br> 바로 앞에서 instanceof를 적용시켰으므로 100% 성공할 것!
4. **입력 객체와 자기 자신의 대응되는 '핵심'필드들의 일치를 하나 하나 확인한다.** <br> 하나라도 다르다면 false를 반환해야 한다.

## 5. 3가지 자문 사항과 주의사항
equals를 다 구현한 다음 3가지를 자문해본다. <br>
**대칭적인가? 추이성이 있는가? 일관적인가?** <br>
3가지에 대해 생각해보고 테스트를 해볼 것. <br>

**주의사항**
- **equals를 재정의할 떈 hashCode도 반드시 재정의 할것** -> 다음ㅇ ㅏ이템
- **너무 복잡하게 해결하려 하지 말고 필드들의 동치성만 확인해도 충분하다.**
- **Object 타입 외를 매개변수로 받는 equals 메서드는 선언해서는 안된다.**
- equals를 테스트를 구현해주는 구글 AutoValue 프레임 워크 사용을 고려해라.



## 요약

꼭 필요한 경우가 아니면 equals를 재정의하지 않는다.** <br>
많은 경우에 Object의 equals가 우리가 원하는 비교를 정확히 해준다! <br>
그래도 재정의 해야할 때는 그 클래스들의 핵심 필드들 모두를 빠짐없이, 다섯 가지 규약을 지켜나가며 비교해야 한다


## Reference
- Effective Java <조슈아 블로크>
