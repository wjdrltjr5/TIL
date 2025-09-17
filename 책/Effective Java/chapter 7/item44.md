## 표준 함수형 인터페이스를 사용하라

---

필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.

java.util.function패키지에는 총 43개의 인터페이스가 담겨있다. 기본 인터페이스 6개만 기억하면 나머지를 충분히 유추해 낼 수 있다.(기본 인터페이스들은 모두 참조 타입용이다.)

| 인터페이스          | 함수 시그니처       | 예                  |
| ------------------- | ------------------- | ------------------- |
| UnaryOperation\<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperation\<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate\<T>       | boolean test(T t)   | Collection::isEmpty |
| Function\<T,R>      | R apply(T t)        | Arrays::asList      |
| Supplier\<T>        | T get()             | Instant::now        |
| Consumer\<T>        | void accept(T t)    | System.out::println |

기본 인터페이스는 기본 타입인 int,long, double용으로 각 3개씩 변형이 생겨난다. 기본 인터페이스의 이름 앞에 해당 기본 타입 이름을 붙여 지였다 (ex. int를 받는 Predicate는 IntPredicate, long을 받아 long을 반환하는 BinaryOperator는 LongBinaryOperator)

유일하게 Function의 변형만 반환 타입이 매개변수화 됐다. LongFunction<int[]>은 long인수를 받아 int[]을 반환한다.

Function인터페이스에는 기본 타입을 반환하는 변형이 9개가 더 있다. 인수와 같은 타입을 반환하는 함수는 UnaryOperation이므로, Function인터페이스의 변형은 입력과 결과의 타입이 항상 다르다.(접두어로 SrcToResult)예로 long을 받아 int를 반환하면 LongToIntFunction이 되는식(총 6개).

나머지(3개)는 입력이 객체 참조이고 결과가 int, long, double인 변형들로 입력을 매개변수화 하고 접두어로 ToResult를 사용한다. 예 ToLongFunction<int[]>은 int[] 인수를 받아 long을 반환한다.

기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있다.BiPredicate<T,U>, BiFunction<T,U,R>, BiConsumer<T,U>이다. BiFunction에는 다시 기본타입을 반환하는 세 변형 ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleFunction<T,U>가 존재한다. Consumer에도 객체 참조와 기본 타입 하나, 즉 인술ㄹ 2개 받는 변형이 OjcDoubleConsmer\<T>, ObjIntConsumer\<T>, ObjLongConsumer\<T>가 존재한다. 이렇게 기본 인터페이스의 인수 2개짜리 변형은 총 9개이다.

BooleanSupplier 인터페이스는 Boolean을 반환하는 Supplier의 변형이다. 이것이 표준 함수형 인터페이스 중 boolean을 이름에 명시한 유일한 인터페이스지만, Predicate와 그변형 4개도 boolean 값을 반환할 수 있다.

표준 함수형 인터페이스 대부분은 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자(item61)

직접만든 함수형 인터페이스에는 항상 @Functional Interface 어노테이션을 사용하라.

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안된다.(item 52 오버로딩은 주의하라)

---

API를 설계할 때 람다도 염두에 두고 설계하라. 입력값과 반환값에 함수형 인터페이스 타입을 활용하라. 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다. 흔치는 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수 도 있음을 잊지말자.
