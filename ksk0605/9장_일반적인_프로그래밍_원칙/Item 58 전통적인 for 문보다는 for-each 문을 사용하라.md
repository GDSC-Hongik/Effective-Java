# Item 58 : 전통적인 for 문보다는 for-each 문을 사용하라

for문 내에서 반복자를 필수로 사용해야 하거나 인덱스를 꼭 사용해야 하는 경우가 아니라면 전통적인 for문은 가독성도 낮추고 변수를 잘못사용하게 할 가능성을 재공한다. 따라서 우리는 왠만한 상황에서는 for-each 문을 애용하는 것이 좋다. 

```java
String[] arr = {"apple", "banana", "orange"};

for (int i = 0; i < arr.length; i++) {
    String fruit = arr[i];
    System.out.println(fruit);
}
```

위 코드에서 어차피 우리는 굳이 인덱스 값이 필요없음에도 전통적인 for문 방식은 한눈에 보기 힘든 코드를 작성해야만 한다. 

```java
String[] arr = {"apple", "banana", "orange"};

for (String fruit : arr) {
    System.out.println(fruit);
}
```

원소만이 필요한 작업에서는 가독성도, 인덱스를 잘못사용해서 찾기 어려운 버그를 만들가능성도 줄이는 for-each문이 좋다.

### 사용할 수 없는 경우

1. 선택한 원소를 제거해야 할 때 
2. 원소 값을 교체해야 할 때 
3. 여러 컬렉션을 같이 순회해야 할 때 (직접 명시적 제어가 필요)

### 결론

전통적인 for문보다 for-each문이 명료하고 유연하고 버그도 예방해준다. 가능한 모든 곳에서 for문 말고 for-each문을 사용하자!