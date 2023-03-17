## 81. wait와 notify보다는 동시성 유틸리티를 애용하라

> 올바르게 wait, notify 를 사용하기 어려우니, 고수준 동시성 유틸리티를 사용하자

- 실행자 프레임워크
- 동시성 컬렉션 (concurrent collection)
- 동기화 장치 (synchronizer)

### 동시성 컬렉션

> 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도 저하

- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 추가한 컬렉션
- 동기화를 내부에서 수행
- 동시성을 무력화하지 못해 원자적으로 묶어 호출 X
    - ‘상태 의존적 수정’ 메서드 추가
    - Map : `putIfAbsent`
        - 주어진 키에 매핑된 값이 아직 없을 때만 새 값을 넣음
        - 기존의 값이 있었다면, 반환 아니면 NULL

        ```java
        // ConcurrentMap으로 구현한 정규화 맵 - 최적은 아니다
        private static final ConcurrentMap<String, String> map =
        				new ConcurrentHashMap<>();
        
        public static String intern(String s) {
        		String previousValue = map.putIfAbsent(s, s);
        		return previousValue == null ? s : previousValue;
        }
        
        // 더 빠른 버전
        public static String intern(String s) {
        		String result = map.get(s); // 검색 기능에 최적화되어 있어
        		if (result == null) { // 없을 때만 putIfAbsent 호출
        				result = map.putIfAbsent(s, s);
        				if (result == null)
        						return s;
        		}
        		return result;
        }
        ```

- `Collections.synchronizedMap` 보다 `ConcurrentHashMap` 을 사용하는 것이 훨씬 좋다
    - 동시성 컬렉션은 낡은 우산이 되어 버림

### 카운트다운 래치

- 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다림
- `CountDownLatch` 의 유일한 생성자는 int 를 받음
    - `countDown` 메서드를 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정
- 다음의 기능을 구현하는데, `wait` 와 `notify` 를 사용하면 어렵지만, 카운트다운을 쓰면 편함
    - 동시성 수준(concurrency)을 매개변수로 받음
        - `CountDown` 을 몇 번 호출해야 대기 중인 스레드를 깨우는지 결정
    - 다수의 작업자 스레드가 동작을 수행
    - 마지막 작업자 스레드가 동작을 수행했을 시 시계를 멈춤

    ```java
    public static long time(Executor executor, int concurrency,
                                Runnable action) throws InterruptedException {
    				// 작업자 스레드가 준비되었음을 타이머 스레드에 통지할 때 사용
            CountDownLatch ready = new CountDownLatch(concurrency);
            CountDownLatch start = new CountDownLatch(1);
            CountDownLatch done = new CountDownLatch(concurrency);
    
            for (int i = 0; i < concurrency; i++) {
                executor.execute(() -> {
                    // 타이머에게 준비를 마쳤음을 알린다.
                    ready.countDown();
                    try {
                        // 모든 작업자 스레드가 준비될 때까지 기다린다.
                        start.await();
                        System.out.println("Start at " + Thread.currentThread().getName());
                        action.run();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    } finally {
                        done.countDown();
                    }
                });
            }
    
            ready.await();  // 모든 작업자가 준비될 때까지 기다린다.
            System.out.println("Ready!");
            long startNanos = System.nanoTime();
            start.countDown();  // 작업자들을 깨운다.
            done.await();   // 모든 작업자가 일을 끝마치기를 기다린다.
            System.out.println("Done!");
            return System.nanoTime() - startNanos;
        }
    ```


### Wait 와 notify 의 주의사항

> 대기 반복문(wait loop) 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 말자.


```java
// wait 메서드를 사용하는 표준 방식
synchronized (obj) {
		while (<조건이 충족되지 않았을 때>)
				obj.wait(); // 락을 놓고, 깨어나면 다시 잡기
		... // 조건 충족 시 동작 수행
}
```

- 새로운 코드라면, 동시성 유틸리티를 사용하자
    - 레거시 데이터를 고려하는 부분이라면, 어쩔 수 없이 사용
- 위 코드와 같이 대기 전 조건을 검사하여 조건이 충족되었다면, `wait` 를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치
    - 만약, 조건이 충족된 상태에서 `notify` 후 `wait` 를 해버리면, 무한 대기 상태로 빠질 수 있음
- 조건이 만족되지 않아도 깨어날 수 있는 상황
    - `notify` 를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
    - 조건이 만족되지 않았지만, `notify` 가 호출되는 경우
    - 대기 중인 스레드 중 일부만 조건을 충족했는데, `notifyAll` 이 호출되는 경우
    - 대기 중인 스레드가 드물게 `notify` 없이 깨어나는 경우 → 허위 각성(spurious wakeup)
- 일반적으로, `notify` 보다는 `notifyAll` 을 사용하는 것이 안전
    - `wait` 는 while문 내부에서 호출하도록 설계