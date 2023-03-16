# Item 59 : 라이브러리를 익히고 사용하라

개발을 할 때, 라이브러리를 잘 이해하고 사용하는 것이 좋다. 우리가 불편했던 것? 왠만하면 선배 개발자들이 이미 고민하고 해결책을 마련해왔다. 우리가 마주하는 수많은 자바 버전들이 바로 그 고민의 결과임을 기억하자. 

```java
// Java의 Math 클래스를 사용하여 원의 넓이를 구하는 메서드
public static double calculateCircleArea(double radius) {
    return Math.PI * Math.pow(radius, 2);
}

// Java의 Random 클래스를 사용하여 임의의 정수를 반환하는 메서드
public static int getRandomInt(int min, int max) {
    Random random = new Random();
    return random.nextInt(max - min + 1) + min;
}

// Java의 Arrays 클래스를 사용하여 배열을 정렬하는 메서드
public static void sortArray(int[] arr) {
    Arrays.sort(arr);
}
```

내가 직접 구현하는 것보다 버그의 가능성도 적고 다른 사람들이 보았을 때에도 알아보기도 더 쉬운 경우가 많으니 누군가 이미 만들어놓았을 가능성을 언제나 생각하고 검색 먼저해보자!

### 결론

왠만하면 내가 만든 것보다 이미 잘 만들어진 것을 사용하는 것이 코스트를 관리할 줄 아는 개발자의 자세