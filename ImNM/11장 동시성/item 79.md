# Item 79 과도한 동기화는 피하라

- 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도 하면 안된다.
- 동기화 영역에서는 가능한 한 일을 적게 하는 것이다.

## 클라이언트에 양도 하면 안된다.

- 동기화된 곳 안에서는 클라이언트가 오버라이딩 ( 재정의 ) 할 수 있는 메서드를 호출 해선 안됨.
- 클라이언트가 넘겨준 함수형 인터페이스도 실행시키면안됨

* 일명 외계인이라고 함 ㅋㅋ

BiConsumer 같은걸로 함수 받아서
본인 리스트를 순회하는 도중에 remove 때려서
ConcurrentModificationException 띄울수도 있음 하지말자.

- 해결하고 싶으면 CopyOnWriteArrayList

```java
private void notifyElementAdded(E element){
  List<SetObserver<E>> snapshot = null;
  synchronized(observers){
    snapshot = new ArrayList<>(observers);
  }
  for (SetObeserver<E> observer : sanpshot)
    observer.added(this, element); // 외계인 메서드가 들어있는 부분을 동기화 바깥으로 제거.
}
```

- 결국 맹점은 동기화 내부에서 예측 불가능하게 비용 낭비 할 수 있는 짓은 하지말자.

- 동기화 영역 안에서 호출 된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야만한다.
- 경쟁하느라 낭비하는 시간이 진?짜 비용이다.
- 병렬로 실행할 기회를 잃고 , 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용.

### 동기화의 문제점.

- StringBuffer -> StringBuilder
  버퍼가 내부적으로 동기화를 계속 씀.
  스트링 빌더는 동기화하징 않은 스트링 버퍼.
- Random -> ThreadLocalRandom
  랜덤도 스레드 안전하게 만들어버림.
