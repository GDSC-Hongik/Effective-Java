## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라.

---

### 싱글턴

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- **클라이언트 테스트하기가 어려워질 수 있다는 단점 존재**
- 첫 번째 방법 : `private` 생성자 + `public static final` 필드

    ```java
    // 생성자는 private으로 감추고, public static 멤버를 통해서만 인스턴스 접근
    // public 필드 방식
    public class Elvis {
        public static final Elvis INSTANCE = new Elvis();
    	
        private Elvis() {}
    }
    ```

    - 장점
        - 간결함
        - 싱글턴임을 API에 들어낼 수 있음
    - 단점
        - 싱글톤을 사용하는 클라이언트 테스트가 어려움
        - 리플렉션으로 `private` 생성자를 호출할 수 있음

            ```java
            // AccessileObject.setAccessible 을 통해 private 생성자 호출 가능
            public static void main(String[] args) {
                try {
                    // getDeclaredConstructor는 getConstructor와 달리 private 호출 가능
                    Constructor<Elvis> defaultconstructor 
                        = Elvis.class.getDeclaredConstructor();
            		
                    // 생성자 접근 제어 허용
                    defaultConstructor.setAccessible(true);
            
                    // 새로운 객체 생성 가능(싱글턴 파괴), hashCode 다름
                    Elvis elvis1 = defaultConstructor.newInstance();
                    Elvis elvis2 = defualtConstructor.newInstance();
            
                } catch (Exception e) {
                    e.printStcakTrace();
                }
            }
            ```

            ```java
            // 리플랙션 공격을 방어하려면, 2번째 객체가 생성될 때 에러 호출
            public class Elvis {
                public static final Elvis INSTANCE = new Elvis();
                private static boolean created;
            	
                private Elvis() {
                    if(created) {
                        throw new UnsupportedOperationException("can't be created by constructor.");
                    }
                    created = true;
                }
            }
            ```

        - **역직렬화할 때 새로운 인스턴스가 생길 수 있음**
            - 디스크에 저장된 데이터를 읽거나 네트워크 통신으로 받은 데이터를 메모리에 쓸 수 있도록 변환

            ```java
            public static void main(String[] args) {
                // 역직렬화 false 출력
                try (ObjectInput in = new ObjectInputstream(new fileInputstream("elvis"))) {
                    Elvis elvis = (Elvis) in.readObject();
                    system.out.println(elvis == Elvis.INSTANCE);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            ```

            ```java
            // 역직렬화 문제를 해결하는 Deserialize 메소드 구현
            public class Elvis implements Serializalbe {
                public static final Elvis INSTANCE = new Elvis();
            	
                // 문법적으로 Override는 아니지만, Deserialize할 때, 이 메소드 사용
                private Object readResolve() {
                    return INSTANCE;
                }
            }
            ```

            - 결국 두 가지 문제점을 해결하면, 간결하다는 장점이 사라짐 → 코드가 난잡해짐
- 두 번째 방법 : `private` 생성자 + 정적 팩터리 메서드
    - 장점
        - 클라이언트 코드 변경 없이 동작 방식을 변경할 수 있음
        - 정적 팩터리를 **제네릭 싱글턴 팩터리**로 만들 수 있음
        - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있음
    - 굳이 다음과 같은 장점이 필요 없다면 `public` 생성자 사용

    ```java
    // 정적 팩터리 방식의 싱글턴
    public class Elvis {
        private static final Elvis INSTANCE = new Elvis();
    	
        private Elvis() {}
    
        public static Elvis getInstance() {
            return INSTANCE;
        }
    }
    ```
    - 원소가 하나뿐인 열거 타입은 싱글턴을 만드는 가장 좋은 방법

    - 세 번째 방법 : 열거 타입(Enum)
    ```java
    public enum Elvis {
        INSTANCE;
    
        public void leaveTheBuilding() {
            ...
        }
    }
    ```
    - 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 리플렉션 공격에서도 방어 가능
    - 단, 상속해야 한다면 사용할 수 없음라