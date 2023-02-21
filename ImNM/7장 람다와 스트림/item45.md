# Item 45 : 스트림은 주의해서 사용하라.

## 핵심정리

- 스트림 파이프라인은 지연 평가 된다.
- 스트림을 과용하면 프로그램이 읽거나 유지보수 하기 어려워진다.
- char 값들을 처리할 때는 스트림을 삼가는 편이 좋음.

### 1. 스트림

스트림은 다량의 데이터 처리 작업을 돕고자 추가됨.
스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻함.
스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

스트림 파이프라인은 소스 스트림 -> 종단 연산으로 끝남
그 사이에 하나 이상의 중간 연산이 있을 수 있다.

- 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠임!!
- 스트림 API는 메서드 체이닝을 지원하는 플루언트 api 임.

```java

try (Scanner s = neww Scanner(dicationary)){
    while(s.hasNext()){
        String word = s.next();
        groups.computeInfAbset(alphabetize(word), (unused) -> new TreeSet<>()).add(word);
    }
}

```

```java

try (Stream<String> words = Files.lines(dicationary)){
   words.collect(groupingBy(word -> alphabetize(ward)))
   .values().stream()
   .filter(group -> gruop.size() >= minGroupSize)
   .forEach(g -> System.out.println(g.size() + ": " g));
}

```

### 2. char 용 스트림?

```java
"Hello ward!".chars().forEach(System.out::print);
// char 가 아닌 int 값을 chars 가 반환됨
// 형변환을 명시적으로 해줘야함!!
```

### 3. 스트림 주의점

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다.
- 람다에서는 final 이거나 사실상 final 인변수만 읽을 수 있고, 지역변수를 수정하는건 불가능하다.
- 코드 블록에서는 return 문을 사용해 빠져나가거나 break, continue 로 반복문을 종료하거나
  반복을 건너뛸수 있지만 스트림은 불가능 하다.

### 4. 스트림 안성맞춤

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합 reduce
- 원소들의 시퀀스를 컬렉션에 모은다 그룹핑
- 원소들의 시퀀스를 특정 조건을 만족하는 원소 찾기
