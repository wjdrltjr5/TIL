## 과도한 동기화는 피하라

---

item78에서 충분하지 못한 동기화의 피해였다면 이번에는 과도한 동기화는 성능을 떨어트리고, 교착상태에 빠트리고, 심지어 예측할 수 없는 동작을 낳기도 한다.

응답불가와 안전 실패를 피할려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다. 예로 동기화된 영역 안에서는 재정의 할 수 있는 메서드는 호출하면 안 되며, 클라이언트가 넘겨준 함수객체(item24)를 호출해서도 안된다. 동기화된 영역을 포함하나클래스 관점에서는 이런 메서드는 모두 바깥 세상에서 온 외계인이다.

예로 어떤 집합Set을 감싼 래퍼 클래스이고, 이 클래스의 클아이언트는 집합에 원소가 추가되면 알림을 받을 수 있다. 관찰자 패턴이다.

```java
//잘못된 코드 동기화 블록 안에서 외계인 메서드를 호출한다.
//리스트의메소드를 써서 그런건가?
public class ObservableSet<E> extends ForwardingSet<E>{
  public ObservableSet(Set<E> set){ super(set);}

  private final List<SetObserver<E>> observers =
          new ArrayList<>();

  public void addObserver(SetObserer<E> observer){
    synchronized(observers){
      observers.add(observer);
    }
  }

  public boolean removeObserver(SetObserver<E> observer){
    synchronized(observers){
      return observser.remove(observer);
    }
  }

  private void notifyElementAdded(E element){
    synchronized(observers){
      for(SetObserver<E> observer : observers){
        observer.added(this, element);
      }
    }
  }

  @Override public boolean add(E element){
    boolean added = super.add(element);
    if(added){
      notifyElementAdded(element);
    }
    return added;
  }

  @Override public boolean addAll(Collection<? extends E> c){
    boolean result = false;
    for(E element : c){
      //이 연산자는 왼쪽 피연산자의 각 비트에 대해 오른쪽 피연산자의 해당 비트와 비트별 OR 연산을 수행하고, 결과를 왼쪽 피연산자에 대입합니다.
      result |= add(element);
    }
    return result;
  }

}
```

관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다. 두경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
@FunctionalInterface public interface SetObserver<E> {
  //ObservableSet에 원소가 더해지면 호출된다.
  void added(ObservableSet<E> set, E element);
}
```

```java
//o~99까지 출력
public static void main(String[] args){
  ObservableSet<Integer> set =
        new ObservableSet<>(new HashSet<>());

  set.addObserver((s, e) -> System.out.println(e));

  for(int i = 0; i < 100; i++){
    set.add(i);
  }
}
```

이때 집합에 추가된 정숫갑을 출력하다가 그 값이 23이면 자기 자신을 제거하는 관찰자를 추가하자.

```java
set.addObserver(new SetObserver<>(){
  public void added(ObservableSet<Integer> s, Integer e){
    if(e == 23){
      s.removeObserver(this);
    }
  }
})
```

이 코드를 실행해 보면 23까지 출력한 다음 ConcurrentModifucationException을 던진다. 관찰자의 added메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다. added메서드는 ObservableSet의 removeObserver을 호출하고 이 메서드는 다시 observers.remove메서드를 호출한다.

여기서 문제가 발생한다. 리스트에서 원소를 제거하려 하는데,마침 지금은 이 리스트를 순회하는 도중이다. 즉 혀용되지 않은 동작 notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록안에 있으므로 동시수정이 일어나지 않도록 보장하지만 자신이 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.

이번에는 구독해지를 하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고 실행자 서비스(ExecutorService, item80)을 사용해 다른 스레드에게 부탁한다

```java
//슬데없이 백그라운드 스레드를 사용하는 관찰자.
set.addObserver(new SetObserver<>(){
  public void added(ObservableSet<Integer> s, Integer e){
    System.out.println(e);
    if(e == 23){
      ExecutorService exec =
              Executors.newSingleThreadExecutor();
      try{
        exec.submit(() -> s.removeObservr(this)).get();
      }catch(ExecutionException | InterruptedException ex){
        throw new AssertionError(ex);
      }finally{
        exec.shutdown();
      }
    }
  }
});
```

이 프로그램을 실행하면 예외는 나지 않지만 교착상태에 빠진다. 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자가 잠그려 시도하지만 락을 얻을 수 없다. 메인스레드가 이미 락을 쥐고 있기 때문 그와 동시에 메인스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다. (데드락상태)

억지스러운 예지만 여기 문제는 자체는 발생 실제로 동기화된 영역 안에서 외계인 메서드를 호출하여 교착상태에 빠지는 사례는 종종있다.

운이좋게도 앞의 두코드는 동기화 영역이 보호하는 자원(관찰자)은 외계인 메서드(added)가 호출될때 일관된 상태였음 그러면 똑같은 상황일때 불변식이 임시로 깨진 상태라면 자바언어의 락은 재진입을 허용하므로 교착상테에 빠지지는 않는다. 예외를 발생시킨 첫 번째 예에서라면 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다. 그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업이 진행중인데도.

재 진입 가능한 락은 객체지향 멀티 스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답 불가(교착상태)가 될 상황을 안전실패로 변모시킬 수 있다.

이런문제는 대부분 어렵지 않게 해결할 수 있다. 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다. notityElementAdded메서드에서라면 관찰자 리스트를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다. 이방식을 적용하면 앞서의 두 예제이서 예외발생과 교착상태 증상이 사라진다.

```java
//외계인 메서드를 동기화 블록 바깥으로 옮겼다.
private void notifyElementAdded(E element){
  List<SetObserver<E>> snapshot = null;
  synchronized(observers){
    snapshot = new ArrayList<>(observers);
  }
  for(SetObserver<E> observer : snapshot){
    //바깥
    observer.added(this, element);
  }
}
```

더 나은 방법은 자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList가 정확이 이 목적으로 설계된 것이다. ArrayList를 구현한 클래스로 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다. 내부의 배열은 절대 수정되지 않으니 순회할 때 락이 필요 없어 매우 빠르다, 다른용도로 쓰인다면 매우 느리겠지만 수정할 일은 드물고 순회만 번번히 일어나는 관찰자 리스트 용도로는 최적이다.

ObservableSet을 CopyOnWriteArrayList를 사용해 다시 구현하면 메서드들은 다음처럼 바꾸면 된다.(add와 addAll메서드는 수정할 게 없다.)

명시적으로 동기화한 곳이 사라졌다.

```java
// CopyOnWriteArrayList를 사용해 구현한 스레드안전하고 관찰 가능한 집합
private final List<SetObserver<E>> observers =
              new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer){
  observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer){
  return observers.remove(observer);
}

private void notifyElementAdded(E element){
  for(SetObserver<E> observer : observers){
    observer.added(this, element)
  }
}
```

기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는것이다.(어디였지 동기화 범위를 최대한 줄이라고 누가 그랬는데)

과도한 동기화가 초래하는 진짜 비용은 경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 진짜 비용이다. 가상머신의 코드 최적화를 제한한다는 점도 과도한 동기화의 또 다른 숨은 비용이다.

가변 클래스를 작성하려거든 두 선택지 중 하나를 따르자

-   동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화 하게 하자

-   동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자(item82) 단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을때에만 사용하자.

java.util은 첫 번째 방식을 취했고, java.util.concurrent는 두 번째 방식을 취했다.(item81)

---

교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지말자. 일반화해 이야기하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자. 가변 클래스를 설계할 떄는 스스로 동기화해야 할지 고민하자, 멀티 코어 세상인 지금은 과도한 동기화를 피하는 게 과거 어느 때보다 중요하다. 합당한 이유가 있을 때만 내부에서 동기화하고, 동기화 했는지 여부를 명확히 밝히자.(item82)
