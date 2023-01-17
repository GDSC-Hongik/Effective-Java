# Item 3 : 생성자나 열거 타입으로 싱글턴임을 보증하라.

### 싱글턴 만드는 법 세가지는 백기선님 더자바 코드를 조작하는 다양한 방법 보세유.

- 객체 직렬화 관련..

### 자바 빈 규약

결국 vo ( Value Object )규약이랑 비슷?

- Getter,Setter( vo 는 아닐듯 )
- Serializable
- equals , hashcode

### Serializable

이게 뭔지 좀 궁금했었음

```java
private static final long serialVersionUID = 1L;
```

구현을 직접 해줄수도, 컴파일러가 컴파일 시점에 집어넣어 줄 수 있음

어디에 쓰는거냐? 클래스 자체에 대한 버져닝과 같음

파일에 썻다가 Book class의 field등이 변경되면
Book class의 버젼이 serialVersionUID 바뀌게 되는거임.
그래서 다시 역직렬화 하는 과정속에서 버젼이 달라서
exception 발생.

```java
public class SerializationExample {

    private void serialize(Book book) {
        try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("book.obj"))) {
            out.writeObject(book);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private Book deserialize() {
        try (ObjectInput in = new ObjectInputStream(new FileInputStream("book.obj"))) {
            return (Book) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
//        Book book = new Book("12345", "이팩티브 자바 완벽 공략", "백기선",
//                LocalDate.of(2022, 3, 21));
//        book.setNumberOfSold(200);

        SerializationExample example = new SerializationExample();
//        example.serialize(book);
        Book deserializedBook = example.deserialize();

//        System.out.println(book);
        System.out.println(deserializedBook);
    }
}

```
