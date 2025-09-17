## 공유 중인 가변 데이터는 동기화해 사용하라.

---

synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.

많은 프로그래머가 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지못하게 막는 요도로만 생각한다.

한객체가 일관된 상태를 가지고 생성되고(item17), 이 객체에 접근하는 메서드는 그 객체에 락을 건다. 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다. 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다 동기화를 제대로 사용한다면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것이다.

동기화에는 중요한 기능이 하나 더 있다. 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다. 동기화는 일관성이 깨진 상태를볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

언어 명세상 long과 double외의 변수를 읽고 쓰는 동작은 원자적이다. 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다. (성능을 높이기 위해 원자적 데이터를 읽고 쓸 때는 동기화 하지 말아야 겠다 라는 생각은 매우 위험하다.)

자바 언어 명세는 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 보이는가는 보장하지 않는다.

동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
이는 한 스레드가 반든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 때문이다.

공유중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패하면 처참한 결과로 이어질 수 있다.

Thread.stop은 사용하지 말자 (자바11이후 제거)

다른 스레드를 멈추는 올바른 방법은 첫번째 스레드는 자신의 boolean필드를 폴링하면서 그 값이 true가 되면 멈춘다. 이 필드를 false로 초기화 해놓고 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다.

boolean필드를 읽고 쓰는 작업은 원자적이라 어떤 프로그래머는 이런 필드에 접근할 때 동기화를 제거하기도 한다.

```java
public class StopThread{
  private static boolean stopRequested;

  public static void main(String[] args) throws InterruptedException{
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      whild(!stopRequested){
        i++;
      }
    });
    backgroundThread.start();
    //TimeUnit 자바에서 제공하는 열거형
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

이프로그램은 1초후 종료되지 않는다. 메인스레드가 1초후 stopRequested를 true로 설정하면서 backgroundThread는 반복문을 빠져나올 것처럼 보이지만 그렇지 않다.

원인은 동기화. 동기화 하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.

```java
// 원래 코드
while(!stopRequested){
  i++
}
//동기화가 빠지면 가상 머신이 이런 최적화를 수행할 수도 있다.
if(!stopRequested){
  while(true){
    i++;
  }
}
```

OpenJDK 서버 vm이 실제로 적용하는 끌어로리기 라는 최적화 기법이다. 이 결과 프로그램은 응답불가 상태가 되어 더이상 진전이 없다. stopRequested필드를 동기화해 접근하면 이 문제르 ㄹ해결할 수 있다.

```java
public class StopThread{
  private static boolean stopRequested;

  private static synchronized void requestStop(){
    stopRequested = true;
  }
  private static synchronized boolean stopRequested(){
    return stopRequested;
  }

  public static void main(String[] args) throws InterruptedException{
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while(!stopRequested()){
        i++;
      }
    });
    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```

쓰기 메서드와 읽기 메서드 모두를 동기화했다 쓰기 메서드만 동기화해서는 충분하지 않다. 쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다!

반복문에서 매번 동기화하는 비용이 크진 않지만 속도가 더 빠른 대안이 있다. stopRequested 필드를 volatile로 선언하면 동기화를 생략해도 된다.

volatile한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게됨을 보장한다.

```java
public class StopThread{
  private static volatile boolean stopRequested;

  public static void main(String[] args) throws InterruptedException{
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while(!stopReuested){
        i++;
      }
    });

    backgroundThread.start();

    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

volatile는 주의해서 사용해야 한다.

```java
//잘못된 코드 동기화가 필요하다.
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber(){
  return nextSerialNumber++;
}
```

이 메서드는 매번 고유한 값을 반환할 의도로 만들어 졌다.

이 메서드의 상태는 nextSerialNumber라는 단 하나의 필드로 결정되는데, 원자적으로 접근할 수 있고, 어떤 값이든 허용한다. 따라서 굳이 동기화 하지 않아도 불변식으로 보호할 수 있어 보인다. 하지만 동기화를 하지 않는다면 동작하지 않는다.

문제는 증가연산자이다. 이 연산자는 코드상으로는 하나지만 실제로는 nextSerialNumber 필드에 두번 접근한다. 먼저 값을 읽고, 하나를 증가한 새로운 값을 저장한다.

만약 두 번째 스레드가 이 두접근 사이를 비집고 들어와 값을 읽어가면 첫번째 스레드와 똑같은 값을 돌려받게 된다. 프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전실패라고한다.

generateSerialNumber메서드에 synchronized 한정자를 붙이면 문제가 해결된다. 동시에 호출해도 간섭하지 않으며 이전 호출이 변경한 값을 읽게됨

가변 데이터는 단일 스레드에서만 쓰도록 하자.

한스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다.

---

여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야한다.

동기화 하지 않으면 한스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다. 공유되는 가변 데이터를 동기화 하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다.(디버깅 난이도 매우 높음 간헐적이거나 특정 타이밍에만 발행할 수도 있고, vm에 따라 현상이 달라지기도 한다.)
