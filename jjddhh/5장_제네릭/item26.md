# 로 타입은 사용하지 말라

### 가장 좋은 오류는 컴파일 오류

```java
// stamp를 담기위한 Collection 선언
private final Collection stamps = ...;

        ...
        
// stamp가 아닌 coin 삽입
stamps.add(new Coin(...));
        
        ...

for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next(); // ClassCastException 발생
    stamp.cancel();    
        }

```

- 런타임에 오류가 발견되는 경우, 문제를 겪는 코드와 원일을 제공한 코드가 물리적으로 떨어져 있을 가능성이 크다. 그렇게 되면 트러블슈팅이 힘들어진다
- 로 타입 사용시 위와 같은 문제가 발생한다
- 매개변수화 타입은 컴파일러에 어떤 타입을 사용할지에 대한 의사를 명확히 전달하는 것으로써 로 타입 사용시 발생하는 문제를 해결한다

### 로 타입을 사용하지 않으면서 타입을 신경쓰고 싶지 않을 때
#### 비한정적 와일드카드 타입(unbounded wildcard type)
- 와일드카드 문자인 '?' 만 사용할 때 비한정적 와일드카드 타입이라 한다
- 비한정적 와일드카드 타입으로 선언된 객체에는 null외의 어떤 원소도 넣을 수 없다

### 한정적 와일드카드 타입 (bounded wildcard type)
- 와일드 카드 문자에 extends 나 super 를 사용한 것
- 비한정적 와일드카드 타입으로는 사실상 추가적인 작업을 하기 힘들다. 따라서 한정적 와일드카드 타입을 사용함으로써 이러한 문제를 해결할 수 있다

[와일드 카드] <br/>
https://snoop-study.tistory.com/113


### 로 타입을 사용하는 예외 상황
- class 리터럴 선언시
- instanceof 연산자 사용시

