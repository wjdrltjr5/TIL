## 다른 타입이 적절하다면 문자열 사용을 피하라

---

문자열(String)은 텍스트를 표현하도록 설계되었다. 하지만 문자열은 워낙 흔하고 자바가 잘 지원해주어 원래 의도하지 않은 용도로 쓰이는 경향이 있다.

문자열은 다른 값 타입을 대신하기에 적합하지 않다. 받은 데이터가 수치형이라면 int, float, BigInteger등 적당한 수치 타입으로 변환해야 한다.

문자열은 열거 타입을 대신하기에 적합하지 않다.

문자열은 혼합 타입을 대신하기에 적합하지 않다.

```java
//혼합 타입을 문자열로 처리한 부적절한 예
String compoundKey = className + "#" + i.next();
```

단점이 많은 방식이다. 혹여라도 두 요소를 구분해주는 문자 #이 두 요소중 하나에서 쓰였다면 혼란스러운 결과를 초래한다. 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다. 절적한 equals, toString, compareTo 메서드를 제공할 수 없으며, String이 제공하는 기능에만 의존해야 한다.

문자열은 권한을 표현하기에 적합하지 않다. 권한을 문자열로 표현하는 경우가 종종있다.

스레드 지역변수 기능을 설계한다고 했을때 그 이름처럼 각 스레드가 자신만의 변수를 갖게 해주는 기능이다.

```java
public class ThreadLocal{
  private ThreadLocal(){}

  //현 스레드의 값을 키로 구분해 저장한다.
  public static void set(String key, Object value);
  // (키가 가리키는) 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```

문제는 스레드 구분용 문자열 키가 전역 이름공간에서 공유된다는 점이다. 이 방식이 의도대로 동작하려면 각 클라이언트가 고유한 키를 제공해야한다. 만약 클라이언트가 서로 소통하지 못해 같은 키를 쓰기로 결정한다면, 의도치 않게 같은 변수를 공유하게 된다. 결국 두 클라이언트 모두 제대로 기능하지 못할 것이다. 보안도 취약하다. 같은키를 사용하여 다른 클라이언트의 값을 가져올 수도 있다.

문자열 대신 위조할수 없는 키를 사용하면 된다. 키를 권한이라고도 한다.

```java
//key클래스로 권한을 구분했다
public class ThreadLocal{
  private ThreadLocal(){}

  public static class Key{
    key(){}
  }

  //위조 불가능한 고유 키 생성
  public static Key getKey(){
    return new Key();
  }

  public static void set(Key key, Object objcet);
  public static Object get(Key key);
}
```

문자열 기반 API의 문제 두가지를 해결해 주지만 아직은 개선할 여지가 있다 set,get은 정적일 필요가 없으니 key클래스의 인스턴스 메서드로 바꾸자 또한 ThreadLocal클래스도 하는 일이 없으니 치우고 중첩클래스 Key의 이름을 ThreadLocal로 바꾸자

```java
public final class ThreadLocal(){
  public ThreadLocal();
  public void set(Object value);
  public Object get();
}
```

이 API에서는 get으로 얻은 Object르르 실제 타입으로 형변환해 써야해서 타입 안전하지 않다

```java
public final class ThreadLocal<T>{
  public ThreadLocal()
  public void set(T value);
  public T get();
}
```

자바의 java.lang.ThreadLocal과 흡사해졌다 문자열 기반 API의 문제를 해결주며 키 기반 API보다 빠르고 우아하다.

---

더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열을 쓰고 싶은 유혹을 뿌리쳐라. 문자열은 잘못 사용하고 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다. 문자열을 잘못 사용하는 예로는 기본타입, 열거타입, 혼합타입이 있다.
