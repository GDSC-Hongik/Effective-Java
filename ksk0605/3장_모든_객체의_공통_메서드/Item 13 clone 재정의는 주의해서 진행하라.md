# Item 13 : clone 재정의는 주의해서 진행하라

- Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스!
    - 메서드가 없음
- clone 메서드는 원본 객체의 필드값과 동일한 값을 가지는 새로운 객체를 생성하고 반환한다.
- 단 이상하게도 clone 메서드는 Cloneable 인터페이스가 아닌 Object에 선언되어있음
    - protected 메서드로 재정의 해야만 사용할 수 있게 선언되어있다.

```java
public class PhoneNumber implements Cloneable{

    ...

    @Override
    public PhoneNumber clone() {
        try{
            return (PhoneNumber) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

위와 같은 방법으로 사용할 수는 있으나 일반적인 **인터페이스의 사용방법과는 차이**가 있다. 아주 이상하다.

### 그렇다고 이러면 끝인가? 그것도 아니다.

Cloneable 구현한 클래스가 불변 객체만을 참조한다면 문제는 없지만 만약 **가변 객체를 참조한다면 원본, 복사본 모두 가변 객체를 참조**하게 되니 이거 아주 위험하다. 

```java
public class Stack {
	private Object[] element;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
    	this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    ...
}
```

복제된 인스턴스가 생성자를 통해 생성된 것이 아니라 참조값을 가져오기 때문에 elements 필드는 원본 인스턴스와 동일한 배열을 참조하게 됩니다. 

따라서 양쪽에서 모두 하나의 element에 관여하게 되고 이는 치명적인 버그를 일으킬 수 있다. 

```java
@Override
public Stack clone() {
	try {
    	Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
    	throw new AssertionError();
    }
}
```

elements 배열의 clone을 재귀적으로 호출해주면 괜찮긴 하다만 element가 final로 선언되어 있으면 불가능한 방식이긴 하다. 

### 그럼 끝인가? 그것도 아니다.

```java
public class HashTable implements Cloneable {
	private Entry[] buckets = ...;
    
    @AllArgsConstructor
    private static class Entry {
    	final Object key;
        Object value;
        Entry next;
    }
    
    @Override
    public HashTable clone() {
    	try {
        	HashTable result = (HashTable) super.clone();
            result.buckets = buckets.clone();
            return result;
        } catch (CloneNotSupportedException e) {
        	throw new AssertionError();
        }
    }
}
```

**원본과 동일한 연결 리스트를 참조하게 되는 경우**에는 충분치 않기에 아래와 같이 연결 리스트까지 복제해야 합니다.

```java
public class HashTable implements Cloneable {
	private Entry[] buckets = ...;
    
    @AllArgsConstructor
    private static class Entry {
    	final Object key;
        Object value;
        Entry next;
        
        Entry deepCopy() {
        	return new Entry(key, value, next == null? null : next.deepCopy());
        }
    }
    
    @Override
    public HashTable clone() {
    	try {
        	HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            
            for (int i = 0; i < buckets.length; i++) {
            	if (buckets[i] != null) {
                	result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e) {
        	throw new AssertionError();
        }
    }
}
```

단, 이 경우도 **재귀 호출로 인해 배열의 길이가 길어지면 스택오버플로우**를 일으킬 수 있다. 

### 그럼 어쩌라고?

### 복사 생성자와 복사 팩터리를 사용하자

자신과 같은 클래스의 인스턴스를 인수로 받는 복사 생성자

```java
public Yum(Yum yum) { ... }
```

복사 생성자를 모방한 정적 팩터리 메서드.

```java
public static Yum newInstance(Yum yum) { ... }
```

이 두 가지 패턴에서는 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 매개변수로 받을 수 있다. 

**단..! 배열은 clone이 좋다.**