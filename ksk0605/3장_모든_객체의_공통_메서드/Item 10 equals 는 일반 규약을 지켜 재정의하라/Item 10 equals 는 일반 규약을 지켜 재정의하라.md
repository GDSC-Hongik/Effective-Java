# Item 10 : equals 는 일반 규약을 지켜 재정의하라

## equals 란?

```java
// Object.java
public boolean equals(Object obj) {
   return (this == obj);
}
```

- Object에 지정되어 두 객체의 같은지 여부에 따라 참/거짓 값을 반환하는 함수
- 모든 객체들이 상속받게 되는 함수이나 각 객체마다 같다고 판정할 기준이 다르기 다르기 때문에 재정의의 필요성이 있다.
    - String 의 equals 재정의 예
    
    ```java
    // String.java
    public boolean equals(Object anObject) {
            if (this == anObject) {
                return true;
            }
            if (anObject instanceof String) {
                String aString = (String)anObject;
                if (!COMPACT_STRINGS || this.coder == aString.coder) {
                    return StringLatin1.equals(value, aString.value);
                }
            }
            return false;
        }
    ```
    
    - String 은 먼저 == 연산자로 같은 레퍼런스 값이면 true를 반환
    - 아닐 경우에는 value 를 비교하기 위해 if 문 안에서 bytecode를 비교하여 동일여부를 판단
    - 이처럼 class에 따라 equals 를 재정의 할 필요성이 있을 수 있음
    - String과 같이 이미 재정의된 equals 를 사용하는 객체가 있을 수 있음

## 재정의하지 않는 경우

- **각 인스턴스가 본질적으로 고유하다.**
    - Integer나 String같은 값을 표현하는 게 아니라 Thread, Service, Controller같은 동작하는 객체 일 때.
- **인스턴스의 ‘논리적 동시성(logical equality)’을 검사할 일이 없다.**
    - 이건 개발자의 맘인데 String과 같이 논리적 동시성 즉, value가 같은지 확인할 필요가 없다면 재정의 할 필요는 없다.
    
    ```java
    public class Student {
    
        // 학생 번호
        private int studentId;
    
        public Student(int studentId) {
            this.studentId = studentId;
        }
    
        // equals 재정의 : 학생 번호가 같으면 true
        @Override
        public boolean equals(Object o) {
            if(o instanceof Student){
                Student student = (Student) o;
                return studentId == student.studentId;
            }
            return false;
        }
    }
    ```
    
- **상위 클래스에서 재정의 한 equals가 하위 클래스에도 딱 들어맞는다.**
- **클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.**
    - 혹여라도 equals가 실행될 수 있으니 아래처럼 아싸리 막아버리는 수도 있겠다.
    
    ```java
    @Override
        public boolean equals(Object obj) {
            throw new AssertionError();
        }
    ```
    

## 재정의 하는 경우

- 객체 식별성(물리적으로 같은 객체인가)이 아니라 논리적 동치성 (값)이 같은지 확인해야할 때, 상위 클래스가 논리적 동치성을 확인할 수 있도록 재정의 되지 않았을 경우
- 값 클래스인 Integer 나 String 은 같은 객체인지 확인하고 싶은게 아니라 같은 값인지 확인하고 싶은 경우가 대부분이니까
    - Map의 키 값 or Set 의 원소 등등
    - 단 값 클래스라고 해도 Enum 과 같은 통제 클래스(언제 어느 인스턴스를 살아 있게 할 지를 통제하는 클래스) 라면 괜찮음.
        - 어차피 똑같은 값을 가지는 다른 인스턴스를 만들어 낼 수 없기 때문
- 재정의 할때는 아래와 같은 규약을 따라야 한다.
    - 안그럼 아주아주 무서운 일이 일어날 것.

## 일반 규약

### 반사성(reflexivity)

**null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.**

- 객체는 자기 자신과 같아야한다.
- 어기기 어렵겠지만 만약 어긴다면 contians같은 함수를 정상적으로 사용하기 어려운 등의 경우가 생길 수 있을 것.

### 대칭성(symmetry)

**null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.** 

- 쉽게 말해 양쪽으로 같은 값이 나와야 한다는 말
- 같은 값이 나오지 않는 예시

```java
public class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

		//대소문자에 관계없이 같은 값을 가지는지 여부를 돌려주는 함수
    @Override
    public boolean equals(Object o ) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)
            return s.equalsIgnoreCase((String) o);
        return false;
    }
}
```

CaseInsensitiveString 인스턴스의 equals는 String을 알기에 의도한대로 작동하지만 String의 equals는 CaseInsensitiveString 를 알지 못하기에 무조건 false를 반환할 것.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("SungKyum");
String s = "sungkyum";
System.out.println(cis.equals(s));
System.out.println(s.equals(cis));
```

실행결과

![스크린샷 2023-01-23 오후 3.49.52.png](../Item%2010%20equals%20%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC/Item%2010%20equals%20%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%E1%84%85%20079cdc69d7ae470b8193515dc10c1bcf/%EA%B2%B0%EA%B3%BC1.png)

- contains 같은 equals를 기반으로 하는 함수에서 의도한 대로 동작하지 않을 수 있고 이것이 찾기 어려운 치명적인 버그가 될 수 있음

```java
//ArrayList 의 contains 안, indexOf 함수가 부르는 indexOfRange 함수의 로직
int indexOfRange(Object o, int start, int end) {
        Object[] es = elementData;
        if (o == null) {
            for (int i = start; i < end; i++) {
                if (es[i] == null) {
                    return i;
                }
            }
        } else {
            for (int i = start; i < end; i++) {
                if (o.equals(es[i])) { // 객체의 equals 로직을 사용
                    return i;
                }
            }
        }
        return -1;
    }
```

- 이러한 경우를 피하기 위해선 결국 동시에 사용하겠다는 것을 포기하는 방법 뿐.
- 위처럼 **상속관계가 아닌 타입이 다른 객체에서 객체의 equals 비교로 true를 반환하는 구현**하는 것은 조심하자!

```java
@Override
public boolean equals(Object o ) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

### 추이성(transitivity)

**null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y)가 true이고  y.equals(z)도 true이면 x.equals(z)도 true이다.** 

- 소크라테스는 사람이다. 사람은 죽는다. 따라서 소크라테스는 죽는다.
- 신경써야 하는 것은 상속관계의 얽힘
    - 특정 클래스를 확장한 클래스는 추이성을 만족시키는 것이 불가능

### case 1 대칭성의 위배

- 하위클래스 쪽에서 상위클래스를 비교하는 결과와 상위클래스 쪽에서 하위 클래스를 비교하는 결과가 다를 수 있음.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

### case 2 추이성 위배

- 대칭성을 만족하게 하더라도 추이성이 위배될 수 있음

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
		// point는 color를 무시하게 한다?
    if (!(o instanceof ColorPoint) )
        return o.equals(this);
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

```java
ColorPoint cp = new ColorPoint(1, 2, Color.white);
Point p = new Point(1,2);
ColorPoint cp2 = new ColorPoint(1,2, Color.black);

System.out.println(cp.equals(p));
System.out.println(p.equals(cp));
System.out.println(p.equals(cp2));
System.out.println(cp.equals(cp2));
```

![스크린샷 2023-01-23 오후 4.34.54.png](../Item%2010%20equals%20%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC/Item%2010%20equals%20%EB%8A%94%20%EC%9D%BC%EB%B0%98%20%EA%B7%9C%EC%95%BD%EC%9D%84%20%EC%A7%80%EC%BC%9C%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%E1%84%85%20079cdc69d7ae470b8193515dc10c1bcf/%EA%B2%B0%EA%B3%BC2.png)

### case 3 리스코프 치환 위배

- 추이성 까지 만족시키기 위해 같은 구현 객체의 비교일때만 true 를 반환하도록 수정한다 하더라도 이것은 리스코프 치환 법칙에 위배된다.

```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

> ***“리스코프 치환 원칙”***

**상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.**
> 
- 상위 클래스의 구현 로직을 하위 클래스가 사용하지 못하게 되는 경우가 생기기 때문

```java
@Getter
class Rectangle {

    private int height;
    private int width;

    public Rectangle(int height, int width) {
        this.height = height;
        this.width = width;
    }
    
    public void setSize(int height, int width) {
        this.height = height;
        this.width = width;
    }
}

@Getter
class Square extends Rectangle {

    public Square(int height, int width) {
        super(height, width);
        if(height != width) {
            throw new AssertionException('cannot create!');
        }
    }
    
    public void setSize(int height, int width) {
        if(height != width) {
            throw new AssertionException('cannot create!');
        }
        this.super(height, width);
    }
}
```

```java
// 직사각형 문맥 로직 수행
public void changeWideSize(Rectangle rec) {
    dim.setSize(dim.getHeight(), dim.getWidth() * 2); // throw Exception.
}
```

다형성에 의해 정사각형이 들어와도 작동하기 때문에 가로/세로 변의 길이가 달라지는 문제 생김 

- 상속대신 컴포지션을 사용하는 방법이 있다.
    
    > **상속이 아닌 private으로 생성된 필드에 인스턴스로 참조하는 방식**
    > 
    
    이 방식은 상속의 메서드 재정의에 의한 문제를 해소할 수 있는 유용한 방법
    
- 추상클래스와 그 구현 방식은 앞선 문제를 야기하지 않는다
    - 상위 클래스를 인스턴스화 할 수 없으니 어찌보면 당연한 이야기

### 일관성(consistency)

**null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.** 

- 가변 클래스이든 불변 클래스이든 상관없이 변할 수 있는 자원을 참조하는 객체라면 언제든지 일관성을 조심해야 한다.
- 잘못된 설계 예시로 java.net.URL의 equals는 네트워크를 통해야하는 IP주소를 비교에 사용하므로 문제가 생김 언제나 **메모리에 존재하는 객체를 기준으로 비교해야 함**
    
    ```java
    public class UrlTest {
        public static void main(String[] args) throws MalformedURLException, UnknownHostException {
            URL googleUrlByDomainName = new URL("https://www.google.co.kr/");
            URL googleUrlByIpAddress = new URL("https://142.250.204.35/"); // 구글의 접속 IP는 다양하므로 테스트때마다 다름
    
            InetAddress address = InetAddress.getByName(googleUrlByDomainName.getHost());
            System.out.println(address.getHostAddress()); //현재 ip주소는 172.217.31.3
    
            InetAddress address2 = InetAddress.getByName(googleUrlByIpAddress.getHost());
            System.out.println(address2.getHostAddress()); // 10분전 ip 주소 142.250.204.35
    
            System.out.println(googleUrlByDomainName.equals(googleUrlByIpAddress)); // false
        }
    }
    ```
    
    시간이 흘러서 DNS서버가 다른 IP주소를 넘겨준다면 false가 나올 수 있음
    
- 객체만이 대상이 아니라 혹은 여러 조건을 동시에 참고하며 비교한다면 문제 생길 수 있음

### null-아님

**null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.**

- o.equals(null) 이 true가 나오는 것을 허용하지 말자

```java
@Override 
public boolean equals(Object o) {
    if (o == null) 
        return false;
    ...
}
```

이거보단 필수 필드를 필연적으로 확인하게 되니

```java
@Override
public boolean equals(Object o ) {
    if (!(o instanceof NullCheck))
        return false;
    NullCheck nc = (NullCheck) o;
    ...
}
```

이렇게 첫번째 피연산자가 null이면 자동으로 false를 반환하도록 하면 좋다. 

### equals 메서드 정의 방법

1. == 연산자를 사용하여 입력이 자기 자신의 참조인지 확인한다. 
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다. 
    1. 올바른 타입이 인터페이스여야 할 경우, 서로다른 구현체끼리도 비교할 수 있도록 메서드 안에서 인터페이스 사용 (다형성) 
3. 입력을 올바른 타입으로 형변한한다. 
4. 입력 객체와 자기 자신의 대응되는 ‘핵심’필드들이 모두 일치하는지 하나씩 검사한다. 
    1. 하나라도 다르면 false
- float 와 double 은 compare 메서드 사용하기
    - 특수 부동소수 값 처리 위해
    - equals 는 오토박싱으로 성능 저하 가능성 있음
- null값을 정상으로 취급하는 참조타입필드는 Objects.equals(Object, Object)로 비교
- CaseInsensitiveString과 같은 복잡한 경우는 표준형 저장후 표준형 비교
- 비교 비용이 싼 것 부터 비교!

> **너무 필요한 상황이 아니면 굳이굳이 재정의 하지 말자!**
>