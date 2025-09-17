## 추상화 수준에 맞는 예외를 던지라.

---

상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다. 이를 예외 번역(exception translation)이라 한다.

```java
// 예외 번역
try{
  ...//저수준 추상화를 이요한다.
}catch(LowerLevelException e){
  // 추상화 수준에 맞게 번역
  throw new HigherLeverException(...);
}
```

AbstractSequentialList에서 수행하는 예외 번역의 예. AbstractSequentialList는 List 인터페이스의 골격 구현(item20)이다. 이 예에서 수행한 예외 번역은 List\<E>인터페이스의 get 메서드 명세에 명시된 필수사항이다.

```java
/**
 * 이 리스트 아느이 지정한 위치의 원소를 반환한다.
 * @throws IndexOutOfBoundsException index가 범위 밖이라면,
 *  발생한다.
*/
public E get(int index){
  ListIterator<E> i = listIterator(index);
  try{
    return i.next();
  }catch(NoSuchElementException e){
    throw new IndexOutOfBoundsException("인덱스 : " + index);
  }
}
```

예외를 번역할 떄, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄(excepion chaining)을 사용하는 게 좋다. 예외 연쇄란 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식이다. 그러면 별도의 접근자 메서드(Throwable의 getCause메서드)를 통해 필요하면 언제든지 저수준 예외를 꺼내 볼 수 있다.

```java
//예외 연쇄
try{
  ..//저수준 추상화를 이용
}catch(LowerLevelException cause){
  throw new HigherLevelException(cause);
}
```

고수준 예외의 생성자는 (예외 연쇄용으로 설계된) 상위 클래스의 생성자에 이 원인을 건내주어 최종적으로는 Throwable(Throwable)생성자 까지 건네지게 한다.

```java
//예외 연쇄용 생성자
class HigherLevelException extedns Exception{
  HighterLevelException(Throwasble cause){
    super(cause);
  }
}
```

대부분의 표준 예외는 예왜 연쇄용 생성자를 가지고 있다. 그렇지 않은 예외라도 Throwable의 initCause 메서드를 이용해 원인을 직접 못박을 수 있다. 예외 연쇄는 문제의 원인을 (getCaused 메서드로) 프로그램에서 접근할 수 있게 해주며 원인과 고수준 예외의 스택 추적 정보를 잘 통합해 준다.

무턱대고 예외를 전파하는 것 보다야 예외 번역이 우수한 방법이지만, 그렇다고 남용해서는 곤란하다. 가능하다면 저수준 메서드가 반드시 성공하도록하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.

아래 계층에서의 예외를 피할 수 없다면 상위 계층에서 그 예외를 조용히 처리하여 문제를 API호출자에까지 전파하지 않는 방법이 있다. 이 경우 발생한 예외는 java.util.logging같은 로깅 기능을 활용하여 기록해두면 조다. 그렇게 해두면 클라이언트 코드와 사용자에게 문제를 전파하지 않으면서 로그를 분석해 추가 조치를 취할 수 있게 해준다.

---

아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란한다면 예외 번역을 사용하라. 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기 좋다.(item75)
