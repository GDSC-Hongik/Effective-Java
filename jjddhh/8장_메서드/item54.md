# null이 아닌, 빈 컬렉션이나 배열을 반환하라

컬렉션이나 배열을 반환할 때, 그 컨테이너가 비어있으면 null 을 반환하는 경우가 있다.

하지만 이렇게 null 을 반환하는 방식은 null 상황을 처리하는 코드를 추가적으로 작성해줘야하기 때문에 번거로워진다.

```java
List<Cheese> cheeses = shop.getCheeses();
if(cheeses != null && cheeses.contains(Cheese.STILTON))
        ...
```

따라서 이런 경우 null 이 아닌 빈 컬렉션이나 배열을 반환해야 한다.

```java
public List<Cheese> getCheeses(){
    return new ArrayList<>(cheesesInStock);    
}
```

위의 코드는 컬렉션이 비어있는 경우에도 빈 컬렉션 할당이 이루어진다. 이를 최적화하면

```java
public List<Cheese> getCheeses(){
    return cheesesInStock.isEmpty() ? 
        Collections.emptyList() : new ArrayList<>(cheesesInStock)    
}
```

이렇게 된다. 컬렉션이 비어있는 경우 이미 생성되어 있는 똑같은 빈 '불변' 컬렉션을 반환하는 것이다.