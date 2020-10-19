# 📇 7장 람다와 스트림

## 아이템 42. 익명 클래스보다는 람다를 사용하라

자바 8 이전까지만해도 우리는 함수 객체를 만들기 위해서 반 강제로 익명 클래스를 쓸 수 밖에 없었다.

```java
Collection.sort(words, new Comparator<String>(){
	public int compare(String s1, String s2){
		return Integer.compare(s1.length(), s2.length());
	}
});
```

이런 추상메서드 하나짜리 인터페이스가 8이상부터는 의미를 인정받고 **람다식(Lambda expression or Lambda)**라고 불리기 시작했다.

```java
Collection.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

다음과 같이 아까는 타입을 명시했지만. 람다를 사용할때는 **"타입을 명시해야 코드가 명확한 경우"**를 제외하고서는 굳이 추가하지 말자. 혹은 컴파일러에서 타입오류를 내뱉는 경우에는 추가해줄 필요가 있다.

또한, 제너릭과 같이 타입추론을 명시하는 경우 람다식은 컴파일러에서 타입 오류를 뱉지 않을 것이다. 

주의 할점이 하나 있는데, **람다는 이름도 없고 문서화도 할 수 없어서, 코드 그 자체로 동작이 명확하게 설명되지 않는 경우에는 아예 쓰지 않는 것이 좋다.** 즉, 람다는 한줄이면 가장 좋고 3줄이 넘어가면 최대한 안쓰는 것이 좋다.

람다는 람다 자신을 참조 할 수 없다. 람다에서 this는 바깥 인스턴스를 가리킨다. 반면에 익명 클래스는 자기 자신을 가리킨다. 그래서 함수에서 자기자신을 가르켜야하는 상황이된다면 반드시 익명 클래스를 사용해야한다. 

이런 이유때문에 람다 자체를 직렬화하는 일은 극히 삼가야한다.



## 아이템43.  람다보다는 메서드 참조를 사용하라

람다보다 더 간결하게 만드는 방식도 있다. 그것은 메서드 참조이다.

```java
map.merge(key, 1, (count, incr)-> count+incr);

map.merge(key, 1, Integer::sum);
```

**물론, 람다가 할 수 없는 일이면 메서드 참조도 불가능하다.** 또한, 메서드 참조가 좋아보이긴하지만, 때에 따라서는 구릴 수도 있다.

```java
service.excute(GoshThisClassNameIsHumongous::action);

service.excute(() -> action()); 
```

  이럴거면 차라리 람다가 괜찮다. 

### 메서드 참조의 유형은 5가지

정적 메서드를 가르키는 메서드 참조, 인스턴스 메서드를 참조하는 경우 

1. 한정적(정적 참조와 비슷) 

   

2. 비한정적 (함수 객체를 적용하는 시점에 전달.) ,  ex) stream 

클래스 생성자,

배열 생성자

| 메서드 참조 유형    | 예시                   | 같은 기능하는 람다                                     |
| ------------------- | ---------------------- | ------------------------------------------------------ |
| 정적                | Integer::parseInt      | str -> Integer.parseInt(str)                           |
| 한정적 (인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now();<br />t-> then.isAfter(t) |
| 비한정적 (인스턴스) | String::toLowerCase    | str->str.toLowCase()                                   |
| 클래스 생성자       | TreeMap<K,V>::new      | () -> TreeMap<k,v>();                                  |
| 배열 생성자         | int[]::new             | len -> new int[len];                                   |



## 아이템 44. 표준 함수형 인터페이스를 사용하라

람다가 생기면서 기존 함수형 매개변수 사례가 생기곤 한다. 물론, 람다를 사용해서 만들기 위해서는 함수형 인터페이스를 구현해야 한다. 근데 이걸 직접 구현하는건 좋지 않다.

**이미 구현되어 있는 표준형 인터페이스를 활용하는 것이 좋다.** 

java.util.function 패키지에는 43개의 인터페이스가 있지만, 이 6개 정도의 인터페이스를 바탕으로 다른 인터페이스를 유추할 수 있다. 

| 인터페이스        | 설명                                                         | 함수 시그니처       | 예시                |
| ----------------- | ------------------------------------------------------------ | ------------------- | ------------------- |
| UnaryOperator<T>  | 인수가 1개인 반환값과 같은 인수를 가진 타입 (parameter = return) | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | 인수가 2개인 반환값과 같은 인수를 가진 타입 (parameter = return) | T apply(T t1, T t2) | BigInteger::add     |
| predicate<T>      | 인수 하나를 받아 boolean으로 반환하는 함수                   | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | 인수와 반환 타입이 다른 경우                                 | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | 인수를 받지 않고, 반환하는 경우                              | T get()             | Instant::now        |
| Consumer<T>       | 인수를 받지만, 반환 값은 없는 경우                           | void accept(T t)    | System.out::println |

이를 바탕으로 여러가지 변형들이 존재한다. Effective Java 3판 265~266 페이지에 거쳐 설명해주고 있다. 물론 위의 6개의 파생이라 대략적으로는 이해하긴 어려움이 없을 것이다.

표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자.

위와 같은 포멧의 비슷한 함수인 Comparator와 같이 비슷한 기능을 가졌지만, 실제로 이것을 표준 함수형 인터페이스로 구현 해야하는 가에 대해서는 Comparator가 가진 특성들을 비교해보면서 직접 짤건지 말건지를 곰곰히 생각하자

- 자주 쓰인다. 이름 자체가 이미 용도 구별이된다.
- 반드시 따라야하는 규약이 존재한다
- 유용한 디폴트 메서드를 제공한다.

그리고 만약에 직접 만든 함수형 인터페이스를 사용할 거라면 `@FunctionalInterface`와 같은 애너테이션을 사용해야한다.