## 인터페이스는 타입을 정의하는 용도로만 사용해라

---

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 달리 말해, 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 애기해 주는 것이다. 인터페이스는 오직 이 용도로만 사용해야 한다.

상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예다

```java
// 잘못 사용한 예
public interface PhysicalConstants{
  // 아보가드로 수(1/몰)
  static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  static final double BOLTZMANN_CONSTANT = 1.380_648_ 52e-23;

  //전자 질량 (kg)
  static final double ELECTRON_MASS = 9.109_383_56E-31;
}

클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라
내부구현에 해당한다.
따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 api로 노출하는 행위다.
```

-   클라이언트 코드가 내부 구현에 해당하는 이 상수들에게 종속되게 함.(이 상수들을 더이상 사용하지 않게 되더라도 바이너리 호환성을 위해 여전히 상수 인터페이스를 구현해야함)
-   final 이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름 공간이 그 인터페이스가 정의한 상수들로 오염되어 버림

---

대신 이러한 방식으로 사용

```java
public class PhysicalConstants {

	private PhysicalConstants() { }

	public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
	public static final double ELECTRON_MASS = 9.109_383_56e-31;

ps. 숫자 리터럴에 사용한 _ 은 숫자값에는 영향을 주지 않고 가독성을 높임
```

---

인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.
