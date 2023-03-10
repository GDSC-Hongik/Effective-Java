# Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라.
JAVA 8의 디폴트 메서드 덕분에, 기존 구현체를 건들이지 않고도 기존 인터페이스에 메서드를 추가할 수 있게 되었다. <br>
엄청 반가운 상황은 아니다, 자바 8 이전의 클래스들은 자기가 구현한 인터페이스에 새로운 메서드가 추가될 일이란 없다고 믿고 살아왔기 때문에, **아무런 합의 없이 메서드를 '삽입' 당한다.** <br>
자바 8에서는 람다를 활용하기 위한 목적으로 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다. <br>
자바 라이브러리의 디폴트 메서드들은 그 품질이 우수하지만, 모든 상황에서 불변식을 해치지 않는 것은 아니다. <br>
예를 들어 자바 8의 Collection 인터페이스에 추가된 removeIf 메서드는 아파치 커먼즈 라이브러리의 SynchronizedCollection을 재정의 하고 있지 않아, 예상하지 못한 동작을 보일 수도 있다. <br> <br>

**디폴트 메서드는 컴파일에 성공할지라도 기존 구현체에 런타임 오류를 일으킬 수 있다.** 흔한 일은 아니지만, 아주 안심할 것도 아니다. <Br>
따라서, 새로운 인터페이스를 만들때는 디폴트 메서드가 아주 유용한 수단이지만, **기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야한다!** <br>
저자는 디폴트 메서드를 적용했다면, 최소 3가지 방식의 테스트를 구현하는 것과 각 인터페이스의 인스턴스를 다양한 작업에 활용하는 클라이언트를 여러개 만들어 볼것을 권한다. <br>



## Reference
- Effective Java <조슈아 블로크>
