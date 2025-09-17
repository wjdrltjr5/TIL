## 옵셔널 반환은 신중히 하라.

---

자바8전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할수 있는 선택지가 2개 있었다. 예외를 던지거나 null을 반환하는 것.

두방법 모두 허점이 있다. 예외는 진짜 예외적인 상황에서만 사용해야 하며(item69) 예외를 생성할 때 스택 추적 전체를 갭처하므로 비용도 만만치 않다 null을 반환할 수 있는 메서드를 호출할 때는(null이 반환될 일이 절대 없다고 확신하지 않는한) 별도의 null처리 코드를 추가해야 한다. null처리를 무시할 경우 추후에 다른곳에서 NullPointerException이 발생할 수 있기 때문.

자바8이 되면서 Optional\<T>이 생겼다 null이아 아닌 T타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.

아무것도 담지 않은 옵셔널은 비었다고 말한다. 반대로 어떤값을 담은 옵셔널은 비지 않았다고 말한다. 옵셔널은 원소 최대 1개 가질 수 있는 불변 컬렉션이다.

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지않아야 할때 T대신 Optioanl\<T>를 반환하도록 선언하면 된다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어진다.

옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다.

```java
//컬렉션에서 최댓값을 구한다.
public static <E extends Comparable<E>> E max(Collection<E> c){
  if(c.isEmpty()){
    throw new IllegalArgumentException("빈 컬렉션");
  }
  E result = null;
  for(E e : c){
    if(result == null || e.compareTo(resutl > 0)){
      resutl = Objects.requireNonNull(e);
    }
  }
  return resutl;
}
```

```java
// 옵셔널 버전
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c){
  if(c.isEmpty()){
    return Optional.empty();
  }
  E resutl = null;

  for(E e : c){
    if(result == null || e.compareTo(resutl > 0)){
      result = Objects.requireNonNull(e);
    }
  }

  return Optional.of(result);
}
```

빈 옵셔녈은 Optional.empty()로 만들고 값이 든 옵셔널은 Optional.of(value)로 생성했다.Optional.of(value)에 null을 넣으면 NullPointerException발생 null값도 허용하는 옵셔널을 만들려면 Optional.ofNullable(value)를 사용하면 된다. 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자.

스트림 종단연산중 상당수가 옵셔널로 반환한다.

```java
//컬렉션에서 최댓값을 구해 Optional<E>로 반환한다 스트림버전
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c){
  return c.stream().max(Comparator.naturalOrder());
}
```

그렇다면 null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준은 무엇인가? 옵셔널은 검사 예외와 취지가 비슷하다.(item71) 즉 반환값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.

```java
//옵셔널 활용 기본값 설정
String lastWordInLexicon = max(words).orElse("단어없음..");
//원하는 예외 던지기
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);

//항상 값이 있다고 가정
Element lastNobleGas = max(Elemnets.NOBLE_GASES).get();
```

이따금 기본값을 설정하는 비용이 아주 커서 부담이 될 때가 있다. 그럴 때는 Supplier\<T>를 인수로 받는 orElseGet을 사용하면 값이 처음 필요할 때 Supplier\<T>를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

앞서의 기본 메서드로 처리하기 어려워 보인다면 API문서를 참조해 filter,map,ifPresent 고급메서드를 검토해보자

여전히 적합한 메서드를 찾지 못했다면 isPresent메서드를 살펴보자 안전밸브 역할의 메서드로 , 옵셔널이 채워져 있으면 true를 비어있으면 false를 반환한다.
신중히 사용해야 한다. 실제로 isPresent를 쓴 코드 중 상당수는 앞에 언급한 메서드로 처리 가능하다.

```java
//부모 프로세스의 ㅍ로세스 ID를 출력하거나, 부모가 없다면
//N/A를 출력 자바9의 ProcessHandle를 사용
Optinal<ProcessHandle> parentProcess = ph.parent();
System.out.println("부모 PID = " + (parentProcess.isPresent() ?
              String.valueOf(parentProcess.get().pid() : "N/A" )));

//--------------------Optional map 사용 ----------
System.out.println("부모 PID = " + ph.parent()
    .map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

스트림을 사용한다면 옵셔널들을 Stream\<Optional\<T>>로 받아서, 그중 채워진 옵셔널들에서 값을 뽑아 Stream\<T>에 건내 담아 처리하는 경우가 드물지 않다

```java
streamOfOptional.filter(Optional::isPresent)
                .map(Optional::get)
```

자바9에서는 옵셔널에 stream()메서드가 추가되었다. 옵셔널에 값이 있으면 그 값을 원소로 담은 스트림으로 값이 없다면 빈 스트림으로 반환한다. 이를 스트림의 flatMap메서드(item45)와 조합하면 앞의 코드를 다음처럼 명료하게 바꿔줄 수 있다.

```java
streamOfOptional.flatMap(Optional::stream);
```

반환값으로 옵셔널을 사용한다고 해서 무조건 득이 되는건 아니다 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.옵셔널을 반환하기 보다는 빈컨테이너를 반환하는 게 좋다(item54)

어떤 경우에 옵셔널로 반환을 해야할까?<br>
결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 옵셔널을 반환한다.

박싱된 기본 타입을 담는 옵셔널은 기본 타입 자체보다 무거울 수 밖에 없다. 값을 두겹이나 감싸기 때문 그래서 전용 옵셔널 클래스를 지원한다. OptionalInt,OptionalLong, OptionalDouble이다 이 옵셔널들도 Optional\<T>가 제공하는 메서들르 거의 다 제공한다. 박싱된 기본타입을 반환하는일은 없도록 하자.

옵셔널을 컬렉션의 키,값,원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다.

---

값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다. 하지만 옵셔널 반환에는 성능 저하가 뒤따르니 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다. 옵셔널을 반환값 이외의 용도로 쓰는 경우는 드물다.
