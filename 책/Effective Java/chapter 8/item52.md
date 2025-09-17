## 다중정의는 신중히 사용하라

---

다음은 컬렉션을 집합, 리스트, 그외로 구분하고자 만든 프로그램이다.

```java
public class CollectionClassifier{
  public static String classify(Set<?> s){
    return "집합";
  }
  public static String classify(List<?> lst){
    return "리스트";
  }
  public static String classify(Collection<?> c){
    return "그 외";
  }
  public static void main(String[] args){
    Collection<?>[] collections = {
      new HashSet<String>(),
      new ArrayList<BigInteger>(),
      new HashMap<String,String>().values()
    };

    for(Collection<?> c : collections){
      System.out.println(classify(c))
    }
  }
}
```

집합, 리스트, 그외를 출력할 것 같지만 그외만 3번 출력한다.
오버로딩의 경우 어느 메서드를 호출할지가 컴파일 타임에 정해지기 때문이다. 컴파일타입에는 for문 안의 c는 항상 Collection<?>타입이다. 런타임에는 타입이 매번 달라지지만 호출할 메서드를 선택하는 데는 영향을 주지 못한다. 따라서 컴파일타임의 매개변수 타입을 기준으로 항상 3번째 메서드가 호출된다.

이처럼 직관과 어긋나는 이유는 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다.

메서드를 재정의했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다.

```java
// 재정의한 메서드 호출 매커니즘
Class Wind{
  String name(){return "포도주";}
}

class SparklingWind extends Wind{
  @Override
  String name(){return "발포성 포도주";}
}

class Champagne extends SparklingWind{
  @Overrid
  String name(){return "샴페인";}
}
public class Overriding{
  public static void main(String[] args){
    List<Wine> wineList = List.of(new Wine(), new SplarklingWine(), new Champagne());

    for(Wine wine : wineList){
      System.out.println(wine.name());
    }
  }
}
```

예상한것처럼 이프로그램은 포도주, 발포성 포도주, 샴페인 을 차례로 출력한다.

for문서에서의 컴파일 타입이 모두Wine 것에 무관하게 가장 하위에서 재정의한 메서드가 실행되는 것이다.

다중정의된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요치 않다 선택은 컴파일타임에, 오직 매개변수의 컴파일타임 타입에 의해 이뤄진다.

다중정의 메서드 코드에서의(맨위 코드) 문제점을 해결하려면 모든 classify메서를 합친후 instanceof로 검사하여 반환하게 한다.

```java
public static String classify(Collection<?> c){
  return c instanceof Set ? "집합" :
            c instanceof List ? "리스트" : "그 외";
}
```

개발자에개는 재정의가 정상적인 동작 반식이고, 다중정의가 예외적인 동작으로 보일 것이다. 즉 재정의한 메서드는 프로그래머가 기대한 대로 동작하지만 다중정의한 메서드는 이러한 기대를 가볍게 무시한다.

다중정의가 혼돈을 일으키는 상황을 피해야한다. 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.(가변인수를 사용하는 메서드라면 다중정의를 아예하지 말자.)

다중정의 하는 대신 메서드 이름을 다르게 지어주는 길도 항상 열려있으니 말이다.

생성자는 이름을 다르게 지을 수 없으니 무조건 다중정의가 된다. 하지만 정적 팩토리라는 대안을 활용할 수 있는 경우가 많다(item1) 또한 생성자는 재정의 할 일이 없으니 헷갈릴 일은 없다.

또한 매개변수가 서로 근본적으로 다르다면 헷갈일이 없다. 두 타입의 값을 서로 어느 쪽으로든 형변환할 수 없다는 뜻.

자바4까지는 모든 기본타입이 모든 참조 타입과 근본적으로 달랐지만, 자바5에서 오토박싱이 도입되면서 달라졌다.

```java
public class SetList{
  public static void main(String[] args){
    Set<Integer> set = new TreeSet<>();
    List<Integer> list = new ArrayList<>();

    for(int i = -3; i < 3; i++){
      set.add(i);
      list.add(i);
    }

    for(int i = 0; i < 3; i++){
      set.remove(i);
      list.remove(i);
    }
    System.out.println(set + "" + list);
  }
}
```

결과는 [-3,-2,-1],[-3,-2,-1]이 아니라 실제로는 집합에서는 음이 아닌 값을 제거하고 리스트에서는 홀수를 제거한후 [-3,-2,-1],[-2,0,2]를 출력한다

set.remove(i)의 시그니처는 remove(Object)이다. 다중정의된 다른 메서드가 없으니 기대한 대로 동작하여 집합에서 0이상의 수들을 제거한다.

한련 list.remove는 다중정의되니 remove(int index)를 선택한다. 리스트의 처음 원소가 [-3,-2,-1,0,1,2]이고 차례로 0번째,1번째,2번째 원소를 제거하면 [-2,0,2]가 남는다.

이문제는 list.remove의 인수를 Integer로 형변환하여 올바른 다중정의 메서드를 선택하게 하면 해결된다. 혹은 Integer.valueOf를 이용해 i를 Integer로 변환한 후 list.remove에 전달해도 된다.

```java
for(int i = 0; i < 3; i++){
  set.remove(i);
  list.remove((Integer) i); //or remove(Integer.valueOf(i))
}
```

이예가 혼란스러웠던 이유는 List<E>인터페이스가 remove(Object)와 remove(int index)를 다중정의했기 때문이다. 제네릭 도입전 자바4까지의 List에는 Object와 int가 근본적으로 달라서 문제가 없었지만 제네릭과 오토박싱이 등장하면서 두 메서드의 매개변수가 근본적으로 다르지 않게 되었다.

자바 언어에 제네릭과 오토박싱을 더한 결과 List 인터페이스가 취약해졌다. 다행히 같은 피해를 입은 API는 거의 없지만, 다중정의시 주의를 기울여야 할 근거로는 충분하다.

자바8에서 도입한 람다와 메서드 참조 역시 다중정의 시의 혼란을 키웠다.

```java
//1번 Thread생성자 호출
new Thread(System.out::println).start();

//2번 ExecutorService의 submit 메서드 호출
ExcutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
```

1번과 2번이 모습은 비슷하지만 2번만 컴파일 오류가 난다. 넘겨진 인수는 모두 System.out::println으로 똑같고 양쪽모두 Runnable을 받는 형제 메서드를 다중정의하고 있다. 원인은 submit다중정의 메서 중에는 Callable\<T>를 받는 메서드도 있다는데 있다.

핵심은 다중정의된 메서드(혹은 생성자)들이 함수형 인터페이스를 인수로 받을때, 비록 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 생긴다는 것

메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안 된다.(서로 다른 함수형 인터페이스라도 서로 근본적으로는 다르지 않다는 뜻)

---

일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋다. 상황에 따라, 특히 생성자라면 이 조언을 따르기가 불가능할 수 있다. 그럴 때는 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야 한다.

기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 만들어야 한다. 그렇지 못하면 프로그래머들은 다중정의된 메서드나 생성자를 효과적으로 사용하지 못할 것이고, 의도대로 동작하지 않는 이유를 이해하지 못할 것이다.
