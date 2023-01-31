# 톱레벨 클래스는 한 파일에 하나만 담으라

- 특정 상황에서는 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라질 수 있다.

```java
public class Main{
    public static void main(String[] args) {
        System.out.println(Utensil.Name + Dessert.Name);
    }
}
``` 

Utensil.java / Dessert.java
```java
class Utensil{
    static final String Name = "pan";
}

class Dessert{
    static final String Name = "cake";
}
```

상황 1. javac Main.java Dessert.java => 컴파일 오류
상황 2. javac Main.java / javac Main.java Utensil.java => pancake 출력
상황 3. javac Dessert.java Main.java => potpie 출력

- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하는 방법을 고려.