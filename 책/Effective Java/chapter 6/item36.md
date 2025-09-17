## 비트 필드 대신 EnumSet을 사용하라

---

열거한 값들이 주로 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴(item34)를 사용해 왔다.

```java
public class Text {
    public static final int STYLE_BOLD =          1 << 0; //1 0001
    public static final int STYLE_ITALIC =        1 << 1; //2 0010
    public static final int STYLE_UNDERLINE =     1 << 2; //4 0100
    public static final int STYLE_STRIKETHROUGH = 1 << 3; //8 1000

    //매개 변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값이다.
    public void applyStyle(int styles) {...}
}
```

이런식으로 비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 바로 비트 필드(bit field)라 한다.

java.util 패키지의 EnumSet클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해 준다. Set인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다. 하지만 EnumSet의 내부는 비트 벡터로 구현되었다. 원소사 총 64개 이하라면, 즉 대부분의 경우에 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.

```java
public ckass Text{
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyle(Set<Style> styles){...}
}
```

```java
//EnumSet은 집합 생성등 다양한 기능의 정적 팩터리를 제공한다.
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));

```

모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이다.(item64)

---

열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다. EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 item34에서 설명한 열거 타입의 장점까지 선사하기 때문이다.
