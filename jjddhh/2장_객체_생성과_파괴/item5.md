## 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 줄 경우
-  싱글턴, 정적 유틸리티 클래스 부적절
   <br/>

따라서 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 '의존 객체 주입' 사용 권장

