## private 생성자나 열거 타입으로 싱글턴임을 보증하라

---

싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

싱글턴을 만드는 방식은 보통 둘 중 하나다. 두 방식 모두 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근 할 수 있는 수단으로 public static 멤버를 하나 마련해둔다.

1. public static 멤버가 final필드인 방식

```java
public class Elvis{

  public static fianl Elvis INSTANCE = new ElVis();

  //생성자는 Elvis.INSTANCE 를 초기화할 때 딱 한번만 호출된다.
  //단 권한이 있는 클라이언트는 리플렉션 API(아이템 65)인 AccessibleObject.setAccessible 을 사용해 private를 호출할 수 있다. 이러한 경우 생성자를 수정하여 두 번 째 객체가 생성되려 할 때 예외를 던지게 한다.
  private Elvis(){}

  public void leaveTheBuildin(){...}

}
```

장점

-   해당 클래스가 싱클턴임이 API에 명백히 드러남
-   간결함

---

2. 정적 팩터리 메서드를 public static 멤버로 제공한다.

```java
public class Elvis{
  private static fianl Elvis INSTANCE = new Elvis();
  private Elvis(){}
  public static Elvis getInstance(){
    return INSTANCE;
  }
  public void leaveTheBuildin(){...}

}
```

장점

-   API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점.
-   원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
-   정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다는 점
    이러한 장점이 필요하지 않다면 public 필드 방식이 좋음.

---

둘중 하나의 방식으로 만든 싱글턴 클래스를 직렬화 하려면 단순이 Serializable을 구현하는 것만으로 부족 모든 인스턴스 필드에 일시적(transient)이라고 선언하고 readResolve메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가 생성됨

3. 열거 타입 방식의 싱글턴 - 바람직한 방법

```java
public enum Elvis{
  INSTANCE;

  public void leaveTheBuilder(){...}

  //public 방식과 비슷하지만 더 간결하고 추가 노력 없이 직렬화할 수 있다
  //대부분상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.
  // 단, 싱글턴이 ENUM외의 클래스를 상속받을 수 없음(주의)
}
```
