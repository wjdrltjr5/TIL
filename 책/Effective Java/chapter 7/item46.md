## 스트림에서는 부작용 없는 함수를 사용하라

---

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다.
이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 순수 함수란 오직 입력만이 결과에 영향을 주는 함수를 말한다.
다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다. 이렇게 하려면 (중간 단계든 종단 단계든) 스트림 연산에 건네는 함수 객체는 모두 부작용이 없어야 한다.

```java
// 스트림 패러다임을 이해하지 못한 채 API만 사용했다 - 따라하지 말것!
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()){
  words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  })
}
```

스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르지만 절대 스트림 코드라 할 수 없다. 스트림 코드를 가장한 반복적 코드이다.

스트림 API의 이점을 살리지 못하여 같은 기능의 반복적 코드보다(조금 더) 길고, 읽기 어렵고, 유지보수에도 좋지 않다. 이 코드의 모든 작업이 종단연산이 forEach 에서 일어나는데 이때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생긴다. forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는것(이 예에서는 람다가 상태를 수정함)을 보니 나쁜 코드이다.

```java
// 스트림을 제대로 활용해 빈도표를 초기화한다.
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()){
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

앞서와 같은 일을 하지만, 이번엔 스트림 API를 제대로 사용했다.
forEach 연산은 종단 연산 중 기능이 가장 적고 가장 덜 스트림 답다. 대놓고 반복적이라 병렬화할 수도 없다 forEach연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자. 물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로도 쓸 수 있다.

수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.

수집기는 총 세가지로

-   toList() 리스트
-   toSet() 집합
-   toCollection(collectionFactory)프로그래머가 지정한 컬렉션 타입

빈도표에서 가장 흔한 단어 10개를 뽑아내는 스트림 파이프 라인을 작성해보자.

```java
//빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인
List<String> topTen = freq.keySet().stream()
        .sorted(comparing(freq::get).reversed())
        .limit(10)
        .collect(toList());
        //toList()는 Collectors의 메서드. Collectors의 멤버를 정적 임포트하여 사용하면 가독성이 좋아진다.
```

sorted에 넘긴 비교자 comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드(item14)이다. 그리고 한정적 메소드 참조이자 여기서 키 추출 함수로 쓰인 freq::get은 입력받은 단어(키) 를 빈도표에서 찾아 그 빈도를 반환한다.

Collectors의 메서드들은 대부분 스트림을 맵으로 취합하는 기능으로 진짜 컬렉션에 취합하는 것보다. 훨씬 복잡하다. 스트림의 각 원소는 키 하나와 값 하나에연관되어 있다. 그리고 다수의스트림 원소가 같은 키에 연관될 수 있다.

가장 간단한 맵 수집기는 toMap(KeyMapper, valueMapper)로, 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.

```java
//toMap 수집기를 사용하여 문자열을 열거 타입 상수에 매핑한다.
private static final Map<String, Operation> stringToEnum =
  Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

toMap형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합
스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IllegalStateException을 던지며 종료될 것이다.

더 복잡한 형태의 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다. 예컨대 toMap에 키 매퍼와 값 매퍼는 물론 병합 함수까지 제공할 수 있다. 병합 함수의 형태는 BinaryOperator\<U>이며, U는 해당 맵의 값 타입이다. 같은 키를 공유하는 값들은 이 병합 함수를 사용해 기존 값에 합쳐 진다. 예컨데 병합 함수가 곱셈이라면 키가 같은 모든 값(키/값 매퍼가 정한다.)을 곱한 결과를 얻는다.

인수3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들때 유용한다.

```java
//다양한 음악가의 앨범들을 담은 스트림을 가지고, 음악가와 베스트앨범을
//연관짓는 일
Map<Artist, Album>topHits = albums.collect(toMap(Album::artis, a -> a, maxBy(comparing(Album::sales))));
```

비교자로는 BinaryOperator에서 정적 임포트한 maxBy라는 정적 팩터리 메소드를 사용했다. maxBy는 Comparator\<T>를 입력받아 BinaryOperator\<T>를 돌려준다. 이경우 비교자 생성 메서드인 comparing이 maxBy에 넘겨줄 비교자를 반환하는데 자신의 키 추출함수로는 Album::sales를 받았다. 즉 앨버 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 짝지은 것이다.

인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는 수집기를 만들 때도 유용하다. 많은 스트림의 결과가 비결정적이다. 하지만 매핑함수가 키 하나에 연결해준 값들이 모두 같을 때, 혹은 값이 다르더라도 모두 허용되는 값일 때 이렇게 동작하는 수집기가 필요하다.

```java
//마지막에 쓴 값을 취하는 수집기
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal);
```

세번째이자 마지막toMap은 네 번째 인수로 맵 팩터리를 받는다. 이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.

이상 세 가지 toMap에는 변종이 있다 그중 toConcurrentMap은 병렬 실행된 후 결과로 ConcurrentHashMap 인스턴스를 생성한다.

Collectors의 groupingBy메서드는 입력으로 분류 함수를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다. 분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다. 이 카테고리가 해당 원소의 맵 키로 쓰인다. 다중정의된 groupingBy중 형태가 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵을 반환한다.반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트다. 알파벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성했다.

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림 수집기도 명시해야 한다.

다운스트림 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다.

이 매개변수를 사용하는 가장 간단한 방법은 toSet()을 넘기는 것이다. 이러면 groupingBy는 는 원소들의 리스트가 아닌 Set값을 가지는 맵을 만든다.

toSet()대신 toCollection(collectionFactory)를 건네는 방법도 있다. 이렇게 하면 리스트나 집합 대신 컬렉션을 값으로 갖는 맵을 생성한다.

다운스트림 수집기로 counting()을 건네는 방법도 있다. 이렇게 하면 각 카테고리(키)를 원소를 담은 컬렉션이 아닌 해당 카테고리에 속하는 워노의 개수(값)와 매핑한 맵을 얻는다.

```java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, Counting()));
```

groupingBy의 세 번째 버전은 다운스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다. 참고로 이 메서드는 점층적 인수 목록 패턴(telescoping argument list pattern)에 어긋난다. 즉, mapFactory 매개변수가 다운스트림 매개변수보다 앞에 놓인다. 이 버전의 groupingBy를 사용하며 맵과 그 안에 담긴 컬렉션의 타입을 모두 지정할 수 있다. ex 값이 TreeSet인 TreeMap을 반환하는 수집기를 만들 수 있다.

이상 총 세가지 groupingBy 각각 대응하는 groupingByConcurrent메서드 들도 볼 수 있다. 이름에도 알 수 있듯 대응하는 메서드의 동시 수행 버전으로 ConcurrentHashMap 인스턴스를 만들어 준다.

많이 쓰이진 않지만 groupingBy의 사촌격인 partitioningBy도 있다. 분류 함수 자리에 프레디키트(predicate)를 받고 키가 boolean인 맵을 반환한다.프레디 키드에 더해 다운스트림 수집기까지 입력받는 버전도 다중정의 되어있다.

counting메서드가 반환하는 수집기는 다운스트림 수집기 전용이다. Stream의 count 메서드를 직접 사용하여 같은 기능을 수행할 수 있으니 collect(counting())형태로 사용할 일은 전혀 없다.

Collections에는 이런 속성의 메서드가 16개나 더있다. 그중 9개는이름이 summing, averaging, summarizing으로 시작하며, 각각 int, long, double 스트림용으로 하나씩 존재한다. 그리고 다중정의된 reducing메서드들, filtering, mapping, flatMpping, collectingAndThen 메서드가 있는데, 대부분의프로그래머는 이들의 존재를 모르고 있어도 상관없다.(설계 관점에서 보면 , 이 수집기들은 스트림 기능의일부를 복제하여 다운스트림 수집기를 더 작은 스트림처럼 동작하게 한 것이다.)

남은 Collections 3개의 메서드는 수집과는 관련이 없다.

minBy와 maxBy는 인수로 받은 비교자를 이용해 스트림에서값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다. Stream 인터페이스의 min과 max메서드를 살짝 일반화 한것이다 BinaryOperator의 minBy와 maxBy 메서드가 반환하는 이진 연산자의 수집기 버전이다.

Collectors의 마지막 메서드는 joining이다. 이 메서드는( 문자열 등의) CharSequence 인스턴스의 스트림에만 적용할 수있다.

매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다.

인수 하나짜리 joining은 CharSequence 타입의 구분문자(delimiter)를 매개변수로 받는다. 연결부위에 이 구분문자를 삽입하는데 구분문자로 쉼표를 입력하면 CSV형태의 문자열을 만들어준다(단, 스트림에 이미 쉽표를 포함한 원소가 있다면 구분문자와 구별되지 않으니 유념하자.)

인수 3개짜리 joining은 구분문자에 더해 접두문자(prefix)와 접미문자(suffix)도 받는다. 접두, 구분, 접미를 각각 '[', ',', ']'로 지정하여 얻은 수집기는 [came, saw, conquered]처럼 마치 컬렉션을 출력한 듯한 문자열을 생성한다.

---

스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. !종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다!.

계산 자체에는 이용하지 말자. 스트림을 올바로 사용하려면 수집기를 잘 알아뒁 한다. 가장 중요한 수집기 택터리는 toList, toSet, toMap, groupingBy, joining이다.
