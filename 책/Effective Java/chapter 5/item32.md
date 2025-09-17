## 제네릭과 가변인수를 함께 쓸 때는 신중하라

---

```java
void test(String ... abc){...};
```

가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데, 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 만들어 진다. 메서드를 선언할 때 실체화 불가 타입으로 가변인수 매개변수를 선언하면 컴파일러가 경고를 보낸다.

```java
//오류 코드
warning: [unchecked] Possible heap pollution from parameterized varang type List<String>
```

매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.

```java
제네릭과 varargs를 혼영하면 타입 안전성이 깨진다!
static void dangerout(List<String> ... stringLists){
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  object[0] = intList; // 힙 오염 발생
  String s = stringLists[0].get(0); // ClassCastException
  타입 안정성이 깨지니 제네릭 varargs배열 매개변수에 값을 저장하는 것은 안전하지 않다.
}
```

@SafeVarargs 어노테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치. 가변인수 메서드를 호출할 때 varargs매개변수를 담는 제네릭 배열이 만들어지는데 메서드가이 배열에 아무것도 저장하지 않고(그 매개변수를 덮어쓰지 않고) 그 배열의 참조가 밖으로 노출되지 않는다면(신뢰할수 없는 코드가 배열에 접근할 수 없다면) 타입안전한다. 즉 이 varargs매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면(varargs의 목적대로만 쓰인다면) 메서드는 안전하다.

```java
자신의 제네릭 매개변수 배열의 참조를 노출한다. - 안전하지 않다!
static <T> T[] toArray(T... args){
  return args;
}
```

이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일 타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다. 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙오염을 이 메서드를 호출한 콜스택으로까지 전이하는 결과를 낳을 수 있다.

```java
static <T> T[] pickTwo(T a, T b, T c){
  switch(ThreadLocalRandom.current().nextInt(3)){
    case 0: return toArray(a, b);
    case 1: return toArray(a, c);
    case 2: return toArray(b, c);
  }
  throw new AssertionError(); // 도달할 수 없다.
}

public static void main(String[] args){
  String[] attributes = pickTwo("좋은", "빠른", "저렴한");

}
```

경고없이 컴파일 되지만 실행하면 ClassCastException을 던진다. pickTwo 메소드의 반환값을 String[] attributes에 저장하기 때문 String[]로 형변환하는 코드를 컴파일러가 자동 생성한다는 점을 놓쳤다. Object[]는 String[]의 하위타입이 아니므로(다운캐스팅 불가) 이 형변환은 실패한다.( 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.)

두 가지 예외가 있다.

-   @SafeVarargs로 제대로 어노테이션된 또 다른 varargs메서드에 넘기는 것은 안전하다.
-   그저 이 배열 내용의 일부 함수를 호출만 하는(varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

```java
제네릭 varargs매개변수를 안전하게 사용하는 메서드
@SafeVarargs
static <T> List<T> flatten(List<? extends T> ... lists){
  List<T> result = new ArrayList<>();
  for(List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

@SafeVarags 어노테이션을 사용해야 할 때를 정하는 규칙은 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @Varargs를 달라. (안전하지 않은 varargs메서드는 절대 작성하지 마라)

-   varargs 매개변수 배열에 아무것도 저장하지 않는다.
-   그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.

아이템28의 조언을 따라 varargs 매개변수를 List 매개변수로 바꿀 수도 있다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists){
  List<T> result = new ArrayList<>();
  for(List<? extends T> list :  lists)
    result.addAll(list);
  return result;
}
```

정적 팩토리 메서드인 List.of를 활용하면 다음 코드와 같이 이 메서드에 임의개수의 인수를 넘길수 있다.(@SafeVarargs어노테이션이 달려있음)

```java
audience = flatten(List.of(friends, romans, countrymen));
```

```java
pickTwo에 적용
static <T> List<T> pickTwo(T a, T b, T c){
  switch(ThreadLocalRandom.current().nextInt(3)){
    case 0: return List.of(a, b);
    case 1: return List.of(a, c);
    case 2: return List.of(b, c);
  }
  throw new AssertionError();
}

public static void main(String[] args){
  List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

결과 코드는 배열없이 제네릭만 사용하므로 타입안전하다.

---

가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입규칙이 서로 다르기 때문이다. 제네릭 varargs매개변수는 타입 안전하지는 않지만, 허용된다. 메서드에 제네릭(혹은 매개변수화된) varargs매개변수를 사용하고 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 어노테이션을 달아 사용하는 데 불편함이 없게끔 하자.
