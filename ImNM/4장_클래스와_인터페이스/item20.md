# Item 20 : 추상클래스보다 인터페이스를 우선하라.

## 인터페이스의 장점

- 자바 8부터 인터페이스도 디폴트 메서드를 제공할 수 있다.
- 기존 클래스도 손쉽게 새로운 인터페이스를 구현해 넣을 수 있다.
- 인터페이스는 믹스인 정의에 안성맞춤이다. ( 선택적인 기능 추가 )
  implements 를 여러번 할 수 있다.
- 계층구조가 없는 타입 프레임워크를 만들 수 있다.
  관계가 명확하지 않은것들 싱어 + 송라이터 = 싱어 송라이터

```java
public interface SingerSongwriter extends Singer, Songwriter{}
```

- 래퍼 클래스와 함께 사용하면 인터페이스는 기능을 향상 시키는 안전하고 강력한 수단이 된다.
  추상 클래스를 상속받으면 얼마든지 상위클래스의 구현에 따라 원하는 동작이 깨질수 있다
  ex hashset addAll 은 내부적으로 add를 사용하기 때문에 추상 클래스를 상속받아 데코레이터와 같이
  로직을껴놓게 되면 상속받은 클래스의 add를 호출 ( 디스 바인딩 ) 때문에 중복해서 들어가게 된다.

- 구현이 명백한 것은 인터페이스의 디폴트 메서드를 사용해 프로그래머의 일감을 덜어 줄 수 있다.

상속은 단일 상속만 가능..

### 자바 8 인터페이스 디폴트 메서드

```java

    static ZoneId getZonedId(String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString + "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
    // 기본으로 구현되어서 이 인터페이스를 구현한 클래스는 쓸 수 있음
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZonedId(zoneString));
    }

```

### 추상 골격 클래스 (skeletal class )

디폴트 메서드로 안되는 것들은 어떻게 해야할까?

- 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다.
- 인터페이스 디폴트 메서드 구현
- 추상 골격 클래스 나머지 메서드 구현
- 템플릿 메서드 패턴

- 다중 상속을 시뮬레이트 할 수 있다.
- 골격 구현은 상속용 클래스이기 때문에 아이템 19를 따라야 한다.

```java
// 추상 골격 클래스를 익명클래스로 구현하는 방식.
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
        // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // 오토박싱(아이템 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // 오토언박싱
                return oldVal;  // 오토박싱
            }

            @Override public int size() {
                return a.length;
            }
        };
    }
```

```java
// 코드 20-2 골격 구현 클래스 (134-135쪽)
public abstract class AbstractMapEntry<K,V>
        implements Map.Entry<K,V> {
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
    @Override public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    // Map.Entry.equals의 일반 규약을 구현한다.
    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(),   getKey())
                && Objects.equals(e.getValue(), getValue());
    }

    // Map.Entry.hashCode의 일반 규약을 구현한다.
    @Override public int hashCode() {
        return Objects.hashCode(getKey())
                ^ Objects.hashCode(getValue());
    }

    @Override public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

- 다중상속처럼 보이게 할 수 있는 방식.
  결국근데 컴포지션이랑 비슷한 느낌인것 같다.

```java
public class MyCat extends AbstractCat implements Flyable {

    private MyFlyable myFlyable = new MyFlyable();

    @Override
    protected String sound() {
        return "인싸 고양이 두 마리가 나가신다!";
    }

    @Override
    protected String name() {
        return "유미";
    }

    public static void main(String[] args) {
        MyCat myCat = new MyCat();
        System.out.println(myCat.sound());
        System.out.println(myCat.name());
        myCat.fly();
    }

    @Override
    public void fly() {
        this.myFlyable.fly();
    }

    private class MyFlyable extends AbstractFlyable {
        @Override
        public void fly() {
            System.out.println("날아라.");
        }
    }
}

```

### 템플릿 메서드 패턴 -> 템플릿 콜백 패턴

- 템플릿 메서드 패턴

```java
public abstract class FileProcessor {

    private String path;

    public FileProcessor(String path) {
        this.path = path;
    }

    public final int process() {
        try(BufferedReader reader = new BufferedReader(new FileReader(path))) {
            int result = 0;
            String line = null;
            while((line = reader.readLine()) != null) {
                result = getResult(result, Integer.parseInt(line));
            }
            return result;
        } catch (IOException e) {
            throw new IllegalArgumentException(path + "에 해당하는 파일이 없습니다.", e);
        }
    }

    protected abstract int getResult(int result, int number);

}
```

- 템플릿 콜백 패턴

```java

   public final int process(BiFunction<Integer,Integer,Integer> operator) {
       try(BufferedReader reader = new BufferedReader(new FileReader(path))) {
           int result = 0;
           String line = null;
           while((line = reader.readLine()) != null) {
               result = operator.apply(result, Integer.parseInt(line));
           }
           return result;
       } catch (IOException e) {
           throw new IllegalArgumentException(path + "에 해당하는 파일이 없습니다.", e);
       }
   }

   // 쓰는법
   fileProcessor.process((a,b)->a+b);
   fileProcessor.process(Integer::sum);

```

### 디폴트 메서드와 Object 메서드

```java
public interface MyInterface {

    default String toString() {
        return "myString";
    }

    default int hashCode() {
        return 10;
    }

    default boolean equals(Object o) {
        return true;
    }
}
```

- 왜 막았을 까?

1.  디폴트 메서드의 용도가 아니다. 인터페이스의 진화가 목적이다.
2.  복잡도를 증가시킨다.

- 클래스가 인터페이스를 이긴다. 인터페이스는 언제까지나 선언이다. 디폴트 메서드도 오버라이딩 가능
- 보다 더 구체적인 인터페이스가 덜 구체적인 인터페이스를 이긴다.

```java

public class MyClass extends Object implements MyInterface {
    // 누가 이길꺼야?
}

```

3. 사실상 토이프로젝트 용이다. 왜 toString hashcode 구현함? 인터페이스 안에 정보가 없음. 어떻게 해쉬코드를 정의할건데.
4. 불안정하다.
   디폴트 메서드로 구현된 toString은 직접 재정의 하지않는 이상 구현클래스 관점에서
   해당 디폴트 메서드가 있는지 없는지 모른다.
   왜냐? 컴파일 에러가 발생하지 않기때문에,
   디폴트메서드는 구현을 필수로 하는 메서드가 아니다.
