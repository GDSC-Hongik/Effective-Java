# Item 2 : 생성자에 매개변수가 많다면 빌더를 고려하라

빌더는 뭐 익히 알고있으니 생소했던거? 다시 챙겨야할거 위주로 정리했습니다.

### 점층적 생성자 패턴

생성자 내부에서 this로 또 다른 생성자 부르기가능...
자바스크립트도 이거랑 같은 전파방법 씀.

```java
// 코드 2-1 점층적 생성자 패턴 - 확장하기 어렵다! (14~15쪽)
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(10, 10);
    }

}

```

### 자바 빈즈 패턴

```java
public class NutritionFacts {
    // 기본값이 있다면, 매개변수들을 기본값으로 초기화된다.
    private final int servingSize  = -1;  // 필수, 기본값 없음
    private final int servings     = -1;  // 필수, 기본값 없음
    private final int calories     = 0;
    private final int fat          = 0;
    private final int sodium       = 0;
    private final int carbohydrate = 0;

    public NutritionFact () {}

    // 세터 메서드들
    public void setServingSize(int val) { servingSize = val;}
    public void setServings(int val) { servings = val;}
    public void setCalories(int val) { calories = val;}
    public void setFat(int val) { fat = val;}
    public void setSodium(int val) { sodium = val;}
    public void setCarbohydrate(int val) { carbohydrate = val;}

}

```

```java
NutritionFacts cocaCola = new NutritionFacts();
// 원하는 매개변수의 값만 설정
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

### 자바 빈즈 패턴에서의 no-arg 생성자 필요

- jpa entity 의 no-arg 생성자가 왜 필요할까?

디비에서 값 불러올때 한방에 객체를 만들 수 없음.

```java
//BigDecimal 컨버터
public class BigDecimalScale6WithBankersRoundingConverter
        implements AttributeConverter<BigDecimal, String> {

    @Override
    public String convertToDatabaseColumn(BigDecimal attribute) {
        if (attribute == null) {
            return BigDecimal.ZERO.toString();
        }
        return attribute.setScale(6, RoundingMode.HALF_EVEN).toString();
    }

    @Override
    public BigDecimal convertToEntityAttribute(String dbData) {
        if (dbData == null) {
            return BigDecimal.ZERO;
        }
        return new BigDecimal(dbData);
    }
}
```

이런식으로 컬럼 하나 하나 넘겨저 오는데
바로 만든다? 쉽지않음

no-arg constructor가 존재해야.
그걸로 객체를 일단 만들고 set으로 필드 하나하나 집어넣어주는거임
자바 빈즈 패턴임!

아이템 1 에서 리플랙션에서
클래스의 타입정보만 있으면 객체는 얼마든지 만들 수 있음.

1. 클래스 타입정보로, no-arg 생성자로 객체생성
2. 필드 정보로 setField 해줌.
