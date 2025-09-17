## 적시에 방어적 복사본을 만들라

---

클라이언트가 여러분의 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍 해야 한다.

어떤 객체든 그 객체의 허락 없이는 외부에서 내부를수정하는일은불가능 하다. 하지만 주의를 기울이지 않으면 자기도 모르게 내부를 수정하도록 허락하는 경우가 생긴다.

```java
//기간을 표현하는 클래스 불변식을 지키지 못함!
public final class Period{
  private final Date start;
  private final Date end;

  /**
   * @param start 시작
   * @param end 종료시각 시작시각보다 위여야 함
   * @throws IllegalArgumentException 시작 시간이 종료 시간보다 늦을때 발생
   * @throws NullPointerException start나 end가 null일때 발생
  */
  public Period(Date start, Date end){
    if(start.compareTo(end) > 0){
      throw new IllegalArugumentEception(start + "가" + end
      + "보다 늦다.");
    }
    this.start = start;
    this.end = end;
  }

  public Date start(){return start;}
  public Date end(){return end;}
  ...
}
```

얼핏 보면 불변식을 지키는 클래스 같지만 Date가 가변이라는 사실을 이용하면 어렵지 않게 불변식을 깨드릴 수 있다.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start,end);
end.setYear(78); //p의 내부를 수정함
```

다행이 자바8이후로는 Date대신 불변인 Instant를 사용하면 된다.(or LocalDateTime, ZonedDateTime도 가능)

Date는 낡은 코드이니 더 이상 사용하면 안된다. 하지만 여전히 옛날 코드에는 잔재가 남아있음 그러므로 외부 공격으로부터 Period인스턴스의 내부를 보호하려면 생성자에 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다.

```java
//방어적 복사를 적용한 수정본
public Period(Date start, Date end){
  this.start = new Date(start.getTime());
  this.end = new Date(end.getTime());

   if(start.compareTo(end) > 0){
      throw new IllegalArugumentEception(start + "가" + end
      + "보다 늦다.");
    }
}
```

새로 작성한 생성자를 사용하면 앞서의 고격은 더이상 위협이 되지 않는다 매개변수의 유효성을 검사(item49)하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한 점에 주목하자. 순서가 부자연스러워 보이지만 반드시 이렇게 작성해야 한다.

멀티 스레딩 환경이라면 원본객체의 유효성을 검사한 후 복사본을 마드는 그 찰나의 취약한 순간에 다른 스레드가 원본 객체를 수정할 위험이 있기 때문이다.(이런 공격을 검사시점/사용시점(time-of-check/time-of-use)공격 혹은 TOCTOU공격이라함)

매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.(Date는 final이 아니므로 clone Date가 정의한 게 아닐 수 있다.)

생성자를 수정하면 앞의 공격은 막아낼 수 있지만 여전히 Period인스턴스는 아직도 변경이 가능하다. 접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문이다.

```java
//Period 인스턴스를 향한 두 번째 공격
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);//p의 내부를 변경했다.
```

가변 필드의 방어적 복사본을 반환하게 함으로 대응

```java
public Date start(){
  return new Date(start.getTime());
}
public Date end(){
  return new Date(end.getTime());
}
```

새로운 접근자 까지 갖추면 Period는 완벽한 불변으로 거듭난다.

Period 자신 말고는 가변 필드에 접근할 방법이 없으니 확실하다. 모든 필드가 객체 안에 완벽하게 캡슐화 되었다.

생성자와 달리 접근자 메서드에는 방어적 복사에 clone을 사용해도 된다. Period가 가지고 있는 Date객체는 java.util.Date임이 확실하기 때문이다.(신뢰할 수 없는 하위클래스가 아님) 하지만 item13의 이유로 인스턴스 복사에는 생성자나 정적 팩토리를 쓰는게 좋다.

매개변수의 방어적 복사 이유는 불변객체를 만들기 위해서만은 아니라 메서드든 생성자든 클라이언트가 제공한 객체의 잠조를 내부의 자료구로에 보관해야 할때 항시 그 객체가 잠재적으로 변경될 수 있는지 생각해야한다.

ex. 클라이언트가 건내준 객체를 내부의Set 인스턴스에 저장하거나 Map인스턴스의 키로 사용한다면 추후 그 개게가 변경될 경우 객체를 담고 있는 Set혹은 Map의 불변식이 깨질것이다.

가변인 내부 객체를 클라이언트에게 반환할 때는 반드시 복사본을 반환해야 한다.(길이가 1이상인 배열은 무조건 가변) 내부에서 사용하는 배열을 클라이언트에 반환해야 할때는 방어적 복사를 잊지말자.

되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다 (item17)

방어적 복사에는 성능저하가 따르고 또 항상 쓸 수 있는 것도 아니다.(같은 패키지에 속하는 등의 이유로) 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다.(그러더라도 문서화를 해놓자.)

다른 패키지에서 사용한다고 해서 넘겨받은 가변 매개변수를항상 방어적으로 복사해 저장해야 하는 것은 아니다.(통제권 이전할때)

---

클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다. 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.
