## 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

---

열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다. 단 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다는 점이다.

대부분의 상황에서 열거 타입을 확장하는건 좋지 않은 생각이다. 확장 타입 원소는 기반 타입 원소로 취급되지만 반대의 경우는 성립하지도 않고 기반타입과 확장타입 원소를 모두 순회할 방법도 딱히 없다. 하지만 확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다. 바로 연산(operation code,opcode) 코드다. 연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다.(item34의 Operation타입도 그중 하나 계산기의 연산기능) 이따금 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가해야 할 때가 있다.

연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다.

```java
public interface Operation{
  double apply(double x, double y);
}

public enum BasicOperation implements Operation{

  PLUS("+"){
    public double apply(double x, double y){return x+y;}
  },
  MINUS("-"){
    public double apply(double x, double y){return x-y;}
  },
  TIMES("*"){
    public double apply(double x, double y){return x*y;}
  },
  DIVIDE("/"){
    public double apply(double x, double y){return x/y;}
  };

  private final String symbol;

  BasicOperation(String symbol){
    this.symbol = symbol;
  }

  @Override
  public String toString(){
    return symbol;
  }
}
```

열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation을 확장해 연산의 타입으로 사용하면 된다.

```java
//확장 가능 열거 타입
public enum ExtendeOperation implements Operation{
  EXP("^"){
    public double apply(double x, double y){return Math.pow(x,y);}
  },
  REMAINDER("%"){
    public double apply(double x, double y){return x % y;}
  };

  private final String symbol;
  ExtendeOperation(String symbol){
    this.symbol = symbol;
  }
  @Override
  public String toString(){
    return symbol;
  }
}
```

```java
public static void main(String[] args){
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}
//클래스가 열거 타입 and Operation하위타입(열거타입이어야 원소 순회가능
// Operation이어야 연산수행 가능)
private static<T extend Enum<T> & Operation> void test
  (Class<T> opEnumType, double x, double y){
    for(Operation op : opEnumType.getEnumConstants())
    System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x,y));
}
```

main 메서드에서는 test 메서드에 ExtendedOperation의 class 리터럴(한정적 타입 토큰 역할item33)을 넘겨 확장된 연산들이 무엇인지 알려준다.

```java
//또 다른 방법
public static void main(String[] args){
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  //한정적 와일드카드 타입(item31)인 Collection<? extends Operation>사용
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y){
  for(Operation op : opSet){
    System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x,y));
  }
}
```

덜 복잡하고 test 메서드가 살짝 더 유연해졌다. 반면, 특정 연산에서는 EnumSet(item36), EnumMap(item37)을 사용하지 못한다.

두 대안프로그램 모두 명령줄 인수로 4와 2를 넣어 실행하면 <br>
4.000000 ^ 2.000000 = 16.000000<br>
4.000000 % 2.000000 = 0.000000<br>
을 출력한다.

열거타입 끼리 구현을 상속할 수 없다는 문제가 있다. 아무상태에도 의존하지 않는 경우에는 디폴트 구현(item20)을 이용해 인터페이스에 추가하는 방법이 있다. 반면Operation 예는 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야만 한다. 이경우에는 중복량이 적으니 문제되지 않지만 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있다.

---

열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 그리고 API가(기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.
