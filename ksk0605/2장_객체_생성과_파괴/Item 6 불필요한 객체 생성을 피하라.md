# Item 6 : 불필요한 객체 생성을 피하라

- 생성비용이 비싼 객체이면 일수록 성능에 영향을 많이 줄 것이다.
- 동일한 일을 하는 코드일지라도 불필요한 객체생성을 최적화 시킨 코드와 아닌 코드는 천지 차이의 성능을 보여준다.
- 오토박싱의 예

```java
public static long sum() {
        Long sum = 0L;
        for (long i = 0; i<=Integer.MAX_VALUE; i++){
            sum += i; // 요기서 무려 2의 31승번의 객체 생성이 이루어진다.
        }
        return sum;
    }

// m1 맥북에어 기준 약 7초 정도 나옴
```