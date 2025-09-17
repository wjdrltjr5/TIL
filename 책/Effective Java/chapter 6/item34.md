## int 상수 대신 열거 타입을 사용하라

---

열거 타입이란 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다. ex) 사계절, 태양계의 해성, 카드게임의 카드 종류 등

```java
public enum Apple{FUJI, PIPPI, GRANNY_SMITH}
public enum Orange{NAVEL, TEMPLE, BLOOD}
```

자바의 열거타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재한다.(싱글턴은 원소가 하나뿐인 열거타입이라 할 수 있고 반대로 열거타입은 싱글턴을 일반화한 형태라 볼 수 있다.)

열거타입에는 각자의 이름 공간이 있어서 이름이 같은 상수도 평화롭게 공존한다. 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수 도 있다. Object메서드(3장)들을 높은 품질로 구현해놨고 Comparable(item14)과 Serializable(12장)을 구현했다.

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.447e7);

    private final double mass;            // 질량(단위: 킬로그램)
    private final double radius;          // 반지름(단위: 미터)
    private final double surfaceGravity;  // 표면중력(단위: m / s^2)

    // 중력상수 (단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() {
        return mass;
    }

    public double radius() {
        return radius;
    }

    public double surfaceGravity() {
        return surfaceGravity;
    }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;
    }
}
```

열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.<br>
ex) Planet.JUPITER.radius() = 7.149E7 출력

열거타입은 근본적으로 불변이라 모든 필드는 final이어야 한다(item17) 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드를 두는게 낫다(item16).

```java
public class WeightTable {
    public static void main(String[] args) {
        double earthWeight = Double.parseDouble("185");
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        for (Planet p : Planet.values()) {
            System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass) );
        }
    }
}
```

열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다. 값들은 선언된 순서로 저장된다. 각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 println과 printf로 출력하기에 안성 맞춤이다.

널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스(item24)로 만든다.

상수마다 동작이 달라져야 하는 상황이라면?

```java
// switch 를 사용한 방법
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

동작은 하지만 예쁘지는 않다. 다행히 열거 타입은 상수별로 다르게 동작하는 코드를 구현하는 더 나은 수단을 제공한다. 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체에 재정의하는 방법이다. 이를 상수별 메서드 구현이라고 한다.

```java
public enum Operation {
    PLUS    {public double apply(double x, double y){return x + y;}},
    MINUS   {public double apply(double x, double y){return x - y;}},
    TIMES   {public double apply(double x, double y){return x * y;}},
    DIVIDE  {public double apply(double x, double y){return x / y;}};

    public abstract double apply(double x, double y);
}
// toString 활용하는 방식
public enum Operation {
    PLUS("+")    {public double apply(double x, double y){return x + y;}},
    MINUS("-")   {public double apply(double x, double y){return x - y;}},
    TIMES("*")   {public double apply(double x, double y){return x * y;}},
    DIVIDE("/")  {public double apply(double x, double y){return x / y;}};

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
    public abstract double apply(double x, double y);
}
```

```java
    // to String이 반환하는 문자열을 해당 열거타입 상수로 변환해주는
    //fromString 메서드 추가
    // 열거 타입 상수 생성 후 정적 필드가 초기화될 때 추가됨.
    private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(Collectors.toMap(Object::toString, e->e));

    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol)); // 주어진 연산이 가리키는 상수가 존재하지 않을 수 있음
    }
```

열거타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다. 이렇게 하려면 컴파일 오류가 발생한다. 열거 타입의 정적 필드 중 열거 타입의 생성자가 실행되는 시점에는 정적 필드들이 초기화 되기 전이라, 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.

한편 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.<br>
급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 생각해보자. 이 열거 타입은 직원의(시간당) 기본 임금과 그날 일한 시간(분 단위)이 주어어지면 일당을 계산해주는 메서드를 갖고있다. 주중에 오버타임이 발생하면 잔업수당이 주어지고, 주말에는 무조건 잔업수당이 주어진다. switch 문을 이용하면 case문을 날짜별로 두어 이 계산을 쉽게 수행할 수 있다.

```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }

}
```

위 코드는 간결하지만 위험한 관리 관점에서는 위험한 코드다. 휴가와 같은 새로운 열거 타입을 추가하려면 그 값을 처리하는 case문을 잊지 말고 쌍으로 넣어줘야 하는 것이다. 자칫 실수하면 휴가기간에 열심히 일해도 평일과 같은 임금을 받는다.

상수별 메서드 구현으로 급여를 정확히 계산하는 방법은 두가지이다.

-   잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다.
-   계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 각 상수가 자신에게 필요한 메서드를 적절히 호출하면 된다.

두방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.<br>
가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 전략을 선택하도록 하는 것이다. 이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연하다.

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY), THURSDAY(WEEKDAY), FRIDAY(WEEKDAY), SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    enum PayType {
        WEEKDAY {
          int overtimePay(int minsWorked, int payRate){
              return minsWorked <= MINS_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
          }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate){
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minsWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        public int pay(int minsWorked, int payRate){
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

}
```

보다시피 switch 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다.하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될수 있다.

```java
//  각연산의 반대 연선을 반환하는 메서드가 필요할때
public static Operation inverse(Operation op) {
  switch (op) {
    case PLUS:      return Operation.MINUS;
    case MINUS:     return Operation.PLUS;
        case TIMES:     return Operation.DIVIDE;
    case DIVIDE:    return Operation.TIMES;
    default: throw new AssertionError("알 수 없는 연산: " + this);
  }
}
```

추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는 게 좋다.

필요한 원소를 컴파일타임에 다알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자. 열거 타입에 정의된 상수 개수가 영원히 고정불변일 필요는 없다.

---

열거 타입은 확실히 정수 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다. 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다. 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch문 대신 상수별 메서드 구현을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.
