## ordinal 인덱싱 대신 EnumMap을 사용하라.

---

이따금 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드(item35)로 인덱스를 얻는 코드가 있다.

```java
class Plant{
  enum LifeCycle{ANNUAL, PERENNIAL, BIENNIAL}

  final String name;
  final LifeCycle lifeCycle;

  Plant(String name, LifeCycle lifeCycle)
}
```

생애주기 기준으로 3개로 구분하여 배열을 이용 각 식물을 해당 집합에 넣는다.

```java
//ordinal()을 배열 인덱스로 사용 - 따라하지 말것!
Set<plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for(int i = 0; i < plantsByLifeCycle.length; i++){
  plantsByLifeCycle[i] = new HashSet<>();
}

for(Plant p : garden){
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
}

//결과 출력
for(int i = 0; i < platnsByLifeCycle.length; i++){
  System.out.printf("%s : %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

동작은 하지만 문제가 한가득이다. 배열은 제네릭과 호환되지 않으니(item28) 비검사 형변환을 수행해야 하고 깔끔히 컴파일 되지 않을 것이다. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다. 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다는 점이다. 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다.(잘못된 값을 사용하면 잘못된 동작을 하거나 운이 좋다면 ArrayIndexOutofBoundsException을 던질 것이다.)

위코드에서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 한다.
그러니 Map을 사용할 수도 있을 것이다.

```java
//EnumMap을 사용해 데이터와 열거 타입을 매핑한다.
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for(Plant.LifeCycle lc : Plant.LifeCycle.values()){
  plantsByLifeCycle.put(lc, new HashSet<>());
}

for(Plant p : garden){
  plantsByLifeCycle.get(p.lifeCycle).add(p);
}
//{ANNUAL=[식물이름,식물이름]}
System.out.println(plantsByLifeCycle);
```

더 짧고 명료하고 안전하고 성능도 원래 버전과 비등하다. 안전하지 않은 형변환을 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블 달 일도 없다. 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 없다.

EnumMap의 성능이 ordinal을 쓴 배열에 비견되는 이유는 그 내부에서 배열을 사용하기 때문에다. 내부 구현 방식을 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻어낸 것이다. EnumMap의 생성사자 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타임 제네릭 타입 정보를 제공한다.(item33)

스트림(item45)을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.

```java
//EnumMap을 사용하지 않는다.
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

```java
//EnumMap을 이용해 데이터와 열거 타입을 매핑했다.
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class), toSet() )))
```

단순한 프로그램에서는 최적화가 굳이 필요 없지만, 맵을 번번히 사용하는 프로그램에는 꼭 필요하다.

스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다. EnumMap버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다. (한해살이와, 여러해살이 식물만 있고 두해살이는 없다면 EnumMap은 3개의 맵, 스트림버전은 2개만 만든다.)

```java
// 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라하지 말것!

public enum Phase {
  SOLID, LIQUID, GAS;

  public enum Transtition{
    MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

    // 행은 from의 ordinal을, 열은 to의 rodinal을 인덱스로 쓴다.
    private static final Transitition[][] TRANSITIONS = {
      {null, MELT, SUBLIME},
      {FREEZE, null, BOIL},
      {DEPOSIT, CONDENSE, null}
    };

    //한 상태에서 다른 상태로의 전이를 반환한다.
    public static Transition from(PHase from, Phase to){
      return TRANSITIONS[from.ordinal()][to.ordinal()];
    }
  }
}
```

컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없음. Phase나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 잘못 수정하면 런타임 오류 발생(ArrayIndexOutOfBoundsException, NullPointerException)하거나 이상하게 동작할 수 있다. 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어날 것이다.

```java
// 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.
public enum Phase {
  SOLID, LIQUID, GAS;

  public enum Transition{
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), BOIL(LIQUID, GAS),
    CONDENSE(GAS, LIQUID), SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

    private final Phase from;
    private final Phase to;

    Transition(Phase from, Phase to){
      this.from = from;
      this.to = to;
    }

    //상전이 맵을 초기화한다.
    private static final Map<Phase, Map<Phase, Transition>> m =
      Stream.of(valuse()).collect(groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class),
       toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));

    public static Transition from(Phase from, Phase to){
      return m.get(from).get(to);
    }
  }
}
```

이 맵의 타입인 Map<Phase, Map<Phase, Transition>>은 "이전 상태에서 '이후 상태에서 전이로의 맵'에 대응시키는 맵"이라는 뜻이다. 이러한 맵의 맵을 초기화하기 위해 수집기(stream.Collecotr)2개를 차례로 준비했다. groupingBy에서는 전이를 이전 상태를 기준으로 묶고, toMap에서는 이후 상태를 전이에 대응 시키는 EnumMap을 생성한다.(x,y) -> y 는 선언만 하고 실제로는 쓰이지 않는다.

위코드에 새로운 상태인 플라스마(PLASMA)를 추가하려면

```java
public enum Phase {
  SOLID, LIQUID, GAS, PLASMA;

  public enum Transition{
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), BOIL(LIQUID, GAS),
    CONDENSE(GAS, LIQUID), SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
//나머지 코드는 수정할 필요 없음
```

---

배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라. 다차원 관계는 EnumMap<..., EnumMap<...>>으로 표현하라. "애플리케이션 프로그래머는 Enum.ordinal()을 웬만해서는 사용하지 말아야 한다.(item35)
