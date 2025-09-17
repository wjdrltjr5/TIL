## 반환 타입으로는 스트림보다 컬렉션이 낫다.

---

스트림은 반복을 지원하지 않는다(스트림 foreach는 결과보고 에만 사용하자). 따라서 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다. API를 스트림만 반환하도록 짜놓으면 반환된 스트림을 for-each로 반복하길 원하는 사용자는 당연히 불만을 토로할 것이다.

(사실 Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 보함할 뿐만 아니라, 정의한 방식대로 동작한다. 그럼에도 for-each로 스트림을 반복할 수 없는 까닭은 Stream이 Iterable을 extends 하지 않아서 이다.)

이문제를 해결줄 멋진 방법은 없다. 얼핏 보면 Stream의 iterator 메서드에 메서드 참조메서드를 건내면 해결될 것 같다. 코드가 좀 지저분하고 직관성이 떨어지지만 못 쓸 정도는 아니다.

```java
// 하지만 이 코드는 컴파일 오류가 난다.
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator){
  ...
}

error: method reference not expected here for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator){}
```

이 오류를 바로잡으려면 메서드 참조를 매개변수화된 Iterable로 적절히 형변환 해줘야 한다.

```java
//스트림을 반복하기 위한 '끔찍한' 우회 방법
for(ProcessHandle ph : (Iterable<ProcessHandle>)
  ProcessHandle.allProcesses()::iterator){
  ...
}
```

작동은 하지만 난잡하고 직관성이 떨어진다. 다행이 어댑터 메서드를 사용하면 상황이 나아진다.

```java
//Stream<E> 를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}
```

어앱터를 사용하면 어떤 스트림도 for-each 문으로 반복할 수 있다.

```java
for(ProcessHandle p : iterableOf(ProcessHandle.allProcesses())){
  ...
}
```

item45의 아나그램 프로그램에서 스트림 버전은 사전을 읽을 때 Files.lines 메서드를 이용했고, 반복 버전은 스캐너를 이용했다. 둘중 파일을 읽는 동안 발생하는 모든 예외를 알아서 처리해준다는 점에서 Files.lines쪽이 더 우수하다. 그래서 이상적으로는 반복 버전에서도 Files.lines를 써야 했다. 이는 스트림만 반환하는 API가 반환한 값을 for-each로 반복하길 원하는 프로그래머가 감수해야 할 부분이다.

반대로 API가 Iterable만 반환하면 이를 스트림 파이프라인에서 처리하려는 프로그래머가 성을 낼 것이다.

```java
//손쉽게 구현한 Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable){
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

객체 시퀸스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주자. 반대로 반환문이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.

하지만 공개 API를 작성할때는 둘 모두를 배려해야한다.

Collection 인터페이스는 Iterable의 하위 타입이고 stream메서드도 제공하니 반복과 스트림을 동시에 지원한다. 따라서 원소 시퀸스를 반환하는 공개API의 반환타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.

Arrays 역시 Arrays.asList와 Stream.of 메서드로 손쉽게 반복과 스트림을 지원할 수 있다. 반환하는 시퀸스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet 같은 표준 컬렉션 구현체를 반환하는 게 최선일 수 있다. 하지만 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀸스를 메모리에 올려서는 안된다.

반환할 시퀸스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션을 구현하는 방안을 컴토해보자.

주어진 집합의 멱집합(한집합의 모든 부부집합을 갖는)을 반환하는 상황이다. {a,b,c}의 멱집합은 {{},{a},{b},{c},{ab},{ac},{bc},{abc}}다. 멱집합의 원소 개수는 2^n 그러니 멱집합을 표준 컬렉션 구현체에 저장하려는 생각은 위험하다. 하지만 AbstractList를 이용하면 휼룡한 전용 컬렉션을 손쉽게 구현할 수 있다.

멱집합을 구성하는 각 원소의 인덱스를 비트 벡터로 사용하는 것이다. 인덱스의 n번째 비트 값은 멱집합의 해당 원소가 원래 집하의 n번째 원소를 포함하는지 여부를 아렬준다. 따라서 0부터 2^n -1 까지의 이진수와 원소n개인 집합의 멱집합과 자연스럽게 매핑된다

```java
//입력 집합의 멱집합을 전용 컬렉션이 담아 반환한다.
public class PowerSetP{
  public static final <E> Collection<Set<E>> of (Set<E> s){
    List<E> src = new ArrayList<>(s);
    if(src.size() > 30){
      throw new IllegalArgumentException("집합에 원소가 너무 많습니다. 최대 30개 :"+ s);
    }
    return new AbstracyList<Set<E>>() {
      @Override
      public int size() {
        return 1 << src.size();
      }

      @Override
      public boolean contains(Object o){
        return o instanceof Set && src.containsAll((Set)o);
      }

      @Override
      public Set<E> get(int index){
        Set<E> result = new HashSet<>();
        for(int i = 0; index != 0; i++, index >>= 1){
          if((index & 1) == 1){
            result.add(src.get(i));
          }
        }
        return result;
      }
    };
  }
}
```

AbstractCollection 을 활용해서 Collection 구현체를 작성할 때는 Iterable용 메서드 외 contains와 size를 구현해야 한다. contatins와 size를 구현하는게 불가능할 때는 컬렉션보다는 스트림이나 Iterable을 반환하는 편이 낫다.(별도의 메서드를 두어 두 방식을 모두 제공해도 된다.)

때로는 단순히 구현하기 쉬운 쪽을 선택하기도 한다. 예컨대 입력 리스트의(연속적인) 부분 리스트를 모두 반환하는 메서드를 작성한다고 했을때 필요한 부분리스트를 만들어 표준 컬렉션에 담는 코드는 3줄이면 충분하다. 하지만 컬렉션은 입력 리스트 크기의 거듭제곱만큼 메모리를차지한다. 멱집합 보다는 낫지만 좋은방법은 아니다.

하지만 입력 리스트의 모든 부분리스트를 스트림으로 구현하기는 어렵지 않다.(약간의 통찰력이 있다면) 첫 번째 원소를 포함하는 부분리스트를 그 리스트의 prefix라 해보자 {a,b,c} 의 경우 {a},{a,b},{a,b,c} 마지막 원소를 포함하는 부분리스트를 suffix라 하자 {a,b,c},{b,c},{c}어떤 리스트의 부분리스트는 단순히 그 리스트의 프리픽스의 서픽스(혹은 서픽스의 프리픽스에) 빈 리스트 하나만 추가하면 된다.

```java
//입력 리스트의 모든 부분리스트를 스트림으로 반환한다.
public class SubLists{
  public static <E> Stream<List<E>> of(List<E> list){
    return Stream.concat(Stream.of(Collections.emptyList()),
    prefixes(list).flatMap(SubLists::suffixes))
  }

  private static <E> Stream<List<E>> prefixes(List<E> list){
    return IntStream.rangeClosed(1, list.size())
            .mapToObj(end -> list.subList(0, end));
  }

  private static <E> Stream<List<E>> suffixes(List<E> list){
    return IntStream.range(0, list.size())
            .mapToObj(start -> list.subList(start, list.size()));
  }
}
```

Stream.concat 메서드는 반환되는 스트림에 빈 리스트를 추가하며, flatMap메서드(item45)는 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다. 마지막으로 프리픽스들과 서픽스들의 스트림은 IntStream.range와 IntStream.rangeClosed가 반환하는 연속된 정숫값들을 매핑해 만들었다.(이 구현은 for반복문을 중첩해 만든 것과 취지가 비슷하다.)

```java
for(int start = 0; start < src.size(); start++){
  for(int end = start +1; end <= src.size(); end++){
    System.out.println(src.subList(start, end));
  }
}
```

이 반복문은 그대로 스트림으로 변환할 수 있다. 그렇게하면 앞서의 구현보다 간결해지지만 읽기에는 더 안 좋을 것이다. 이방식의 취지는 item45 데카르트 곱용 코드와 비슷하다.

```java
//입력 리스트의 모든 부분리스트를 스트림으로 반환한다.
public static <E> Stream<List<E>> of(List<E> list){
  return IntStream.range(0, list.size())
          .mapToObj(start -> IntStream.rangeClosed(start + 1, list.size()).mapToObj(end -> list.subList(start,end)))
          .flatMap(x -> x);
}
```

바로 앞의 for반복문처럼 이코드도 빈 리스트는 반환하지 않는다. 이부분을 고치려면 앞처럼 concat을 사용하거나 rangeClosed 호출 코드의 1을(int)Math.signum(start)로 고쳐주면 된다.

스트림을 반환하는 두가지 구현을 알아봤는데, 모두 쓸만은 하지만 반복을 사용하는게 더 자연스러운 상황에서도 사용자는 그냥 스트림을 쓰거나 Stream을 Iterable로 변환해주는 어댑터를 사용해야 한다. 하지만 어탭터는 코드를 어수선하게 만들고 더 느리다.

---

원소 시퀸스를 반환하는 메서드를 작성할 때는, 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고, 양쪽을 다 만족시키려 노력하자.

컬렉션을 반환할 수 있으면 그렇게 하라. 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList같은 표준 컬렉션이 담아 반환하라. 그렇지 않으면 앞서의 멱집합 예처럼 전용 컬렉션을 구현할지 고민하라. 컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라. 만약 나중에 Stream인터페이스가 Iterable을 지원하도록 자바가 수정된다면, 그때는 안심하고 스트림을 반환하면 될 것이다.
