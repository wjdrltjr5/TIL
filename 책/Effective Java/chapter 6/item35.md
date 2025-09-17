## ordinal 메서드 대신 인스턴스 필드를 사용하라

---

대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.

```java
//ordinal 메서드를 잘못 사용한 예
public enum Ensemble{
  SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTTET, NONET, DECTET;

  public int numberOfMusicians(){return ordinal() + 1};
}
```

위코드는 동작은 하지만 유지보수하기가 끔찍한 코드 상수선언을 바꾸는 순간 오동작하며, 이미 사용 중인 정수와 값이 같은 상수는추가할 방법이 없다. 또한 값을 중간에 비워둘 수도 없다. 중복값 사용도 불가.

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하자.

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

enum api문서에는 ordinal 메소드를 대부분의 프로그래머는 사용할 일이 없다고 적혀있다. 이메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다.
