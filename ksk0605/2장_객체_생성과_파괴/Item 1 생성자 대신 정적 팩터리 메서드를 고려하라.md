# Item 1 : 생성자 대신 정적 팩터리 메서드를 고려하라

- 전통적인 클래스 인스턴스를 반환하도록 하는 방법은 public 생성자
    
    ```java
    // 생성자
    public Boolean(boolean value) {
            this.value = value;
    }
    ```
    
- 하지만 아래와 같은 **정적 팩터리 메서드 방식**을 고려해야 한다.
    - 클래스의 인스턴스를 반환하는 단순한 static 메서드
    
    ```java
    
    public static final Boolean TRUE = new Boolean(true);
    
    public static final Boolean FALSE = new Boolean(false);
    
    // 정적 팩터리 메서드
    public static Boolean valueOf(boolean b) {
            return (b ? TRUE : FALSE);
    }
    ```
    

## static factory method가 생성자에 비해 가지는 장점

- 이름을 가질 수 있다
    - 위 예시의 valueOf함수와 같이 다양한 이름을 설정 가능
    
    ```java
    new BigInteger(int, int, Random);
    
    BigInteger.probablePrime;
    // 이쪽이 소수인 BigInteger를 반환한다 라는 의미를 더 잘 전달
    ```
    
- 호출될 때 마다 인스턴스를 새로 생성하지는 않아도 된다.
    - 불필요한 객체의 생성을 막을 수 있음.
    - 인스턴스의 통제가 가능
        - 싱글턴 패턴 구현가능
        - 인스턴스화 불가
        - 등 인스턴스가 단 하나임을 보장할 수 있도록 할 수 있음
        
        ```java
        public class LottoNumber {
            private static final int MIN_LOTTO_NUMBER = 1;
            private static final int MAX_LOTTO_NUMBER = 45;
        
            private static Map<Integer, LottoNumber> lottoNumberCache = new HashMap<>();
        
            static {
                IntStream.range(MIN_LOTTO_NUMBER, MAX_LOTTO_NUMBER)
                        .forEach(i -> lottoNumberCache.put(i, new LottoNumber(i)));
            }
        
            private int number;
        
        		// 생성자로 직접 인스턴스화 하지 못하도록 함
            private LottoNumber(int number) {
                this.number = number;
            }
        
            public LottoNumber of(int number) {  // LottoNumber를 반환하는 정적 팩토리 메서드
                return lottoNumberCache.get(number);
            }
        ...
        }
        
        // 출처 : https://tecoble.techcourse.co.kr/post/2020-05-26-static-factory-method/
        ```
        
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
    
    ```java
    class Hero {
    	public static Hero of(String job){
    		if (job == "전사") {
    			return new Warrior();
    		} else if (job == "마법사") {
          return new Wizard();
        } else {
          return new Archer();
        }
    	}
    }
    ```
    
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
    
    ```java
    // 원소의 수에 따라 두 가지 하위 클래스 중 하나를 반환하는 EnumSet
    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
    ```
    
- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
    - 이게 뭔소리야?

## static factory method의 단점

- 상속해줄 클래스는 무조건 public이나 protected 생성자가 있어야 하기 때문에 정적 팩터리 메서드만 제공하면 상속을 할 수가 없다.
    - 위 로또  넘버 클래스는 상속 불가능
    - 무조건 단점이라기보단 제약을 걸어줄 수 있다는 장점으로 볼 수도!
- 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
    - 딱히 생성자처럼 명확한게 아니라서 일일이 찾아봐야한다는 것?
    - 그래도 of, valueOf, getInstance 등등 흔히 사용되는 이름들은 있다.