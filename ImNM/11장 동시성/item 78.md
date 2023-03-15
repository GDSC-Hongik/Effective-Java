# Item 78 공유 중인 가변 데이터는 동기화해 사용하라

- 동기화는 배타적 실행 뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
- Thread.stop은 사용하지말어 ( 자바 11에서 완전히 제거 )
- 쓰기와 읽기가 모두 동기화 되지 않으면 동작을 보장하지 않는다.

## 동기화

- 객체를 하나의 일관된 상태에서 다르 ㄴ일관된 상태로 변화시킨다.
- 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종결과를 보게 해준다.
- 다시말하면, 한 스레드가 만든 변화를 다른 스레드에서 확인 못함.

- 자바 언어 명세는 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장하지만
- 한 스레드가 저장한 값이 다른 스레드에게 `보이는가` 는 보장하지 않는다!!!!!!

### 제공방식

1. polling 으로 상태 받아오기.

```java
public class StopThread {
  // 애초에 공유자원에 스레드의 스탑 여부를 적용시켜버림.
  private static boolean stopRequested;

  public static void main(String[] args) throws InterruptedExcetion {
    Thread backgroundThread = new Thread(()->{
      int i = 0 ;
      // hoisting 에 의해 최적화를 시켜버림.
      while (!stopRequested)
        i++;
    });
    // 백그라운드 쓰레드 시작
    backgroundThread.start();
    // 카운트 치면서 1초뒤에 종료대기를 기대 ( main 쓰레드 슬립. )
    TimeUnit.SECONDS.sleep(1);
    // 1초뒤에 stopRequested true 됨.
    stopRequested = true;
  }
}

```

- 동기화가 빠지면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수없다.
- 즉 가성머신이 알아서 최적화를 해버린다 왜?
  위에서말한 한 스레드가 저장한 값이 다른 스레드에게 `보이는가` 는 보장하지 않기 때문에
  덧붙여서 캐시 되어서 변경된게 제대로 확인 안될수 도 있는 거고.

```java
// hoisting 되면서 최적화 되어버려 무한으로 while 에 빠짐.
if(!stopRequsetd)
  while (true)
    i++;
```

2. 읽기와 쓰기 모두 동기화를 보장하라.

```java
private static synchronized void requestStop() {
  stopRequested = true;
}
private static synchronized boolean stopRequested() {
  return stopRequested;
}
```

3. volatile

배타적 수행 ( 락 )과는 관련없지만
항상 최근에 기록된 값을 읽게 됨을 보장.

```java
private static volatile int nextSerialNumber = 0;
```

### volatile 의 위험성

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
  return nextSerialNumber++;
}
```

- 뭐가 문제임?

증가 연산자 에 있음 ( ++ )
코드상으로는 하나지만 실제로는 nextSerialNubmer 필드에 두번 접근한다.
먼저 값을 읽고 , 그런다음 새로운 값을 저장.
두 번째 쓰레드가 이 사이를 비집고 들어오면 결국 똑같음 동시성 보장이 안됨.

- 결국 atomicLong 쓰삼.
