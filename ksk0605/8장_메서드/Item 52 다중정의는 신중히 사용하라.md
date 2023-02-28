# Item 52 : 다중정의는 신중히 사용하라

## 다중정의와 재정의의 차이

다중정의의 가장 큰 장점은 다양한 타입의 인자를 받아들일 수 있다는 점.  다중정의를 남용하게 되면 가독성이 떨어는 것 뿐 아니라 의도대로 동작하지 않아서 디버깅이 어려워 질 수 있다. 

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

출력결과는 **‘그 외’**만 3번이다. 

반면 아래의 **재정의 예시**를 보자 

```java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
```

이때는 의도대로 **‘포도주’, ‘발포성 포도주’ , ‘샴페인’** 순서대로 잘 출력한다.

이 둘의 차이는 **재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문**이다. 다시말하면 재정의한 메서드는 런타임 중에 클래스에 의해 선택되지만 다중정의는 컴파일하는 순간에 선택된다는 것. 

## 그럼 언제 다중정의를 사용해?

다중정의는 다음과 같은 경우에 고려해볼 수 있다.

- 같은 이름의 함수나 메서드가 여러 타입의 인자를 받아들여야 할 때
- 같은 기능을 하는 함수나 메서드를 다양한 인자 타입에 대해 제공해야 할 때

그러나 인자 타입이나 개수가 크게 다르지 않은 경우, 오버로딩 대신에 인자를 다르게 하는 함수나 메서드를 따로 정의하는 것이 코드 가독성을 높여준다.  (**정적 팩터리 메서드**나 아예 **이름이 다른 메서드**로)

### ObjectOutputStream 클래스

```java
....
    public void writeBoolean(boolean val) throws IOException {
        bout.writeBoolean(val);
    }

    /**
     * Writes an 8 bit byte.
     *
     * @param   val the byte value to be written
     * @throws  IOException if I/O errors occur while writing to the underlying
     *          stream
     */
    public void writeByte(int val) throws IOException  {
        bout.writeByte(val);
    }

    /**
     * Writes a 16 bit short.
     *
     * @param   val the short value to be written
     * @throws  IOException if I/O errors occur while writing to the underlying
     *          stream
     */
    public void writeShort(int val)  throws IOException {
        bout.writeShort(val);
    }

    /**
     * Writes a 16 bit char.
     *
     * @param   val the char value to be written
     * @throws  IOException if I/O errors occur while writing to the underlying
     *          stream
     */
    public void writeChar(int val)  throws IOException {
        bout.writeChar(val);
    }

    /**
     * Writes a 32 bit int.
     *
     * @param   val the integer value to be written
     * @throws  IOException if I/O errors occur while writing to the underlying
     *          stream
     */
    public void writeInt(int val)  throws IOException {
        bout.writeInt(val);
    }

    /**
     * Writes a 64 bit long.
     *
     * @param   val the long value to be written
     * @throws  IOException if I/O errors occur while writing to the underlying
     *          stream
     */
    public void writeLong(long val)  throws IOException {
        bout.writeLong(val);
    }

    /**
     * Writes a 32 bit float.
     *
     * @param   val the float value to be written
     * @throws  IOException if I/O errors occur while writing to the underlying
     *          stream
     */
    public void writeFloat(float val) throws IOException {
        bout.writeFloat(val);
    }
....
```

꼭 오버로딩이 아니어도 가독성을 높일 수 있는 방법이 있음을 명시하자. 

## 생성자의 경우에는?

생성자는 이름을 다르게 지을 수 없으니 정적 팩터리 메서드를 활용하는 방법이 있겠지만 여러 생성자가 같은 수의 매개변수를 받는 상황을 완전히 피할 수는 없을 것이다. 

매개변수 수가 같은 다중정의 메서드가 많더라도, 그 중 어느 것이 주어진 매개변수 집합을 처리할지 명확히 구분된다면 헷갈릴 일은 없을 것이다. 이는 매개변수중 하나 이상이 **‘근본적으로 다르다**’라는 뜻.  즉, 두 타입이 서로 어느 쪽으로든 형변환 할 수 없으면 된다는 것. 이 조건을 충족하면 어느 다중정의 메서드를 호출할지가 매개변수들의 런타임 타입만으로 결정된다. 따라서 컴파일 타임 타입에는 영향을 받지 않게 되고, 혼란을 주는 주된 원인이 사라진다. 

## 주의해야하는 몇 가지 예시

### 오토박싱

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
        }

        for (int i = 0; i < 3; i++) {
            set.remove(i);
        }
        System.out.println(set);
    }

}
```

위 코드는 문제없이 [-3, -2, -1, 0, 1, 2] 의 값에서 [0, 1, 2]가 제거되어 [-3, -2, -1]가 출력되지만 

```java
public class SetList {
    public static void main(String[] args) {
       List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            list.remove(i);
        }
        System.out.println(list);
    }

}
```

위 코드에서는 왜인지 [-2, 0, 2]가 출력된다. 이는 List의 remove가 다중정의 되어있기 때문인데 List.remove(Object)와 List.remove(int) 중 후자가 선택되며 벌어지는 일.

이는 자바 5에서 오토박싱이 추가되면서 int와 Integer 클래스가 근본적으로는 다르지 않게 되었고 List인터페이스가 취약해진 결과이다. 

### **람다와 메서드 참조의 혼란**

```java
// 1번. Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번. ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

1번은 정상적으로 작동하나 아래에서는 컴파일 오류가 발생하는데 그 이유는 submit 다중 정의 메서드 중에는 Callable<T>를 받는 메서드도 있고, void를 반환하는 메서드도 있지만 println이 다중정의 되어있기 때문에 정확히 어떤 걸 참조하고 있는지 알 수 없기 때문.

```java
...
    public void println() {
        newLine();
    }

    /**
     * Prints a boolean and then terminate the line.  This method behaves as
     * though it invokes {@link #print(boolean)} and then
     * {@link #println()}.
     *
     * @param x  The {@code boolean} to be printed
     */
    public void println(boolean x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }

    /**
     * Prints a character and then terminate the line.  This method behaves as
     * though it invokes {@link #print(char)} and then
     * {@link #println()}.
     *
     * @param x  The {@code char} to be printed.
     */
    public void println(char x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }
...
```

실제 println 다중 정의를 보면 알 수 있듯 System.out::println은 부정확한 메서드 참조가 되고 결국 에러를 뱉어내게 되는 것. (양쪽 다 다중정의 되어있기에 발생하는 문제) 

## 결론

일반적으로 매개변수 수가 같을 때에는 다중정의를 피하자. 생성자의 경우에는 헷갈릴 만한 매개변수는 형변환 하여 정확한 다중정의 메서드가 선택되도록 하자. 이것이 불가능하다면 (ex, 기존 클래스가 새로운 인터페이스를 구현해야하는 상황) 기존의 메서드들은 모두 동일하게 동작하도록 수정하자.