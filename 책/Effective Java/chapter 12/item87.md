## 커스텀 직렬화 형태를 고려해보라

---

먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.

기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야한다.

객체의 물리적 표현과 논리적 표현이 같다면 기본 직렬화 형태라도 무방하다.

```java
//기본 직렬화 형태에 적합한 후보
public class Name implements Serializable{
  /**
   * 성. null이 아니어야 함.
   * @serial
  */
  private final String lastName;

  /**
   * 이름. null이 아니어야 함.
   * @serial
  */
  private final String firstName;

  /**
   * 중간이름. 중간이름이 없다면 null
  */
  private final String middleName;
}
```

성명은 논리적으로 이름, 성, 중간이름이라는 3개의 문자열로 구성되며, 앞 콛의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다.

기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다. 앞의 Name 클래스의 경우에는 readObject 메서드가 lastName과 firstName필드가 null이 아님을 보장해야 한다.

다음 클래스는 직렬화 형태에 적합하지 않은 예.

```java
// 기본 직렬화 형태에 적합하지 않은 클래스
public final class StringList implements Serializable{
  private int size = 0;
  private Entry head = null;

  private static class Entry implements Serializable{
    String data;
    Entry next;
    Entry previous;
  }
}
```

논리적으로는 이 클래스는 일련의 문자열을 표현한다.
물리적으로는 문자열들을 이중 연결 리스트로 연결했다.
객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 네가지 면에서 문제가 생긴다

-   공개API가 현재의 내부표현 방식에 영구히 묶인다. 연결리스트를 사용하지 않게 되더라도 제거할 수 없다.
-   너무 많은 공간을 차지할 수 있다. 직렬화 형태는 연결 리스트의 모든 엔트리와 연결 정보까지 기록했지만, 엔트리와 연결 정보는 내부 구현에 해당하니 직렬화 형태에 포함할 가치가 없다. 이처럼 직렬화 형태가 너무 커져서 디스크에 저장하거나 네트워크로 전송하는 속도가 느려진다.
-   시간이 너무 많이 걸릴 수 있다. 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수 밖에 없다.
-   스택 오버플로를 일으킬 수 있다. 기본 직렬화 과정은 객체 그래프를 재귀순회하는데 중간 정도 크기의 객체 그래프에서도 자칫 스택오버플로를 일으킬 수 있다.

StringList의 합리적인 직렬화 형태
단순히 리스트가 포함된 문자열의 개수를 적은다음 그 뒤로 문자열들을 나열하는 수준이면 충분

```java
//합리적인 커스텀 직렬화 형태를 갖춘 StirngList
public final class StringList implements Serializable{
  private transient int size = 0;
  private transient Entry head = null;

  //이제는 직렬화 되지 않는다.
  private static class Entry{
    String data;
    Entry next;
    Entry previous;
  }

  //지정한 문자열을 이 리스트에 추가한다.
  public final void add(String s){...}

  /**
   * 이 {@code StringList} 인스턴스를 직렬화 한다.
   *
   * @serialData 이 리스트의 크기(포함된 문자열의 개수)를
   * 기록한 후 ({@code int}), 이어서 모든 원소를(각각은 {@code
   * String}) 순서대로 기록한다.
   */
  private void writeObject(ObjectOutputStream s)throws IOException{
    s.defaultWriteObject();
    s.writeInt(size);

    //모든 원소를 올바른 순서로 기록한다.
    for(Entry e = head; e != null; e = e.next){
      s.writeObject(e.data);
    }
   }
   private void readObject(ObjectInputStream s)throws IOException, ClassNotFoundException{
    s.defaultReadObject();
    int numElements = s.readInt();

    //모든 원소를 읽어 이 리스트에 삽입한다.
    for(int i = 0; i < numElements; i++){
      add((String) s.readObject());
    }

    ...//코드 생략
   }
}
```

StringList의 필드 모두가 transient 더라도 writeObject와 readObject는 각각 먼저 defaultWriteObject와 defaultReadObject를 호출한다. 클래스 인스턴스필드 모두가 transient면 호출하지 않아도 된다고 들었을지 모르지만, 직렬화 명세는 이 작업을 무조건 하라고 요구한다.

이렇게 해야 향후 릴리스에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호(상위,하위)호환되기 떄문이다.

기본 직렬화를 수용하던 하지않던 defaultWriteObject메서드를 호출하면 transient를 제외한 인스턴스 필드가 직렬화 된다. 따라서 transient로 선언해도 되는 인스턴스 필드에는 모두 붙여야 한다.

해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient한정자를 생략해야 한다.

커스텀 직렬화형태를 사용한다면 StringList예에서처럼 대부분의 인스턴스 필드를 transient로 선언해야 한다.

기본 직렬화를 사용한다면 transient필드들은 역 직렬화될 때 기본값으로 초기화됨을 잊지말자 null, 0, false 기본값을 그대로 사용해서는 안된다면 readObject 메서드에서 defaultReadObject를 호출한 다음, 해당 필드를 원하는 값으로 복원하자(item88) 혹은 그값을 처음 사용할 떄 초기화 하는 방법도 있다.(item83)

기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야 한느 동기화 메커니즘을 직렬화에도 적용해야 한다.

모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면 writeObject도 synchronized로 선언해야 한다.

어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자. UID가 일으키는 잠재적 호환성 문제 해결을 위해

무작위 long값

구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.

---

자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고 그렇지 않다면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라.

직렬화 형태도 공개 메서드(item51)를 설계할 때에 준하는 시간을 들여 설계해야한다.
