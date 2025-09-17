## wait와 notify보다는 동시성 유틸리티를 애용하라.

---

wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.

java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다. 바로 실행자 프레임워크, 동시성 컬렉션(concurrent collection), 동기화 장치(synchronizer)다

동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다(item79)

동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다. 그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 상태 의존적 수정 메서드들이 추가되었다. 이 메서드들은 자바8에서는 일반 컬렉션 인터페이스에도 디폴트 메서드형태로 추가되었다.

예로 Map이 putIfAbsent(key, value)메서드는 주어진 키에 매핑된 값이 아직 없을 때만 새 값을집어 넣는다. 그리고 기존값이 있었다면 가 값을 반환하고 없었다면 null을 반환한다. 이 메서드 덕에 스레드 안전한 정규화 맵을 쉽게 구현할 수 있다.

```java
//String.intern의 동작을 흉내 내어 구현한 메서드
//ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아님
private static final ConcurrentMap<String, String> map = new ConcurrentHashMap<>();
// ConcurrentMap은 get같은 검색기능에 최적화 되어있어 먼저
//get을 호출하여 필요할떄만 putIfAbsent를 호출하면 더 빠르다.
public static String intern(String s){
  String result = map.get(s);
  if(result == null){
    result = map.putIfAbsent(s,s);
    if(resutl = null){
      result = s;
    }
  }
  return result;
}
```

ConcurrenHashMap은 동시성이 뛰어나며 속도도 무척 빠르다.

Collections.synchronizedMap 보다는 ConcurrentHashMap을 사용하는게 훨씬 좋다. 동기화된 맵을 동시성 맵으로 교체하는 것만으로 동시성 애플리케이션의 성능은 극적으로 개선된다.

컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다. Queue를 확장한 BlockingQueue에 추가된 메서드 중 take는 큐의 첫원소를 꺼낸다 이때 큐가 비어 있다면 새로운 원소가 추가될 때까지 기다린다.

이런 특성 덕에 BlockingQueue는 작업 큐(생산자-소비자큐)로 쓰기에 적합하다. 작업 큐는 하나 이상의 생산자(producer)스레드가 큐에 있는 작업(work)을 큐에 추가하고, 하나 이상의 소비자(consumer)스레드가 큐에 있는 작업을 꺼내 처리하는 형태다. ThreadPoolExecutor를 포함한 대부분의 실행자 서비스(item80)구현체에서 BlockingQueue를 사용한다.

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다. 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore다. CyclicBarrier와 Exchanger는 그보다 덜 쓰인다 가장 강력한 동기화 장치는 Phaser다

카운드다운래치는 일회성 장벽으로 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다. CounDownLatch의 유일한 생성자는 int값을 받으며, 이값이 래치의 countDown메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지를 결정한다.

CountDownLatch를 사용한 코드(wait,notify를 사용한것 보다 훨씬 간단)

```java
//동시 실행 시간을 재는 간단한 프레임워크
public static long time(Executor executor, int concurrency,
 Runnable action) throws InterruptedException{
  CountDownLatch ready = new CountDownLatch(concurrency);
  CountDownLatch start = new CountDownLatch(1);
  CountDownLatch done = new CountDownLatch(concurrency);

  for(int i = 0; i < concurrency; i++){
    executor.execute(() -> {
      //타이머에게 준비르르 마쳤음을 알린다.
      ready.countDown();
      try{
        // 모든 작접자 스레드가 준비될 때까지 기다린다.
        start.await();
        action.run();
      }catch(InterruptedException e){
        Thread.currentThread().interrupt();
      }finally{
        //타이머에게 작업을 마쳤음을 알린다.
        done.countDown();
      }
    })
  }

  ready.await();//모든 작업자가 준비될 때 까지 기다린다.
  long startNanos = System.nanoTime();
  start.countDown();;// 작업자들을 깨운다.
  done.await();//모든 작업자가 일을 끝마치기를 기다린다.
  return System.nanoTime() - startNanos;
 }
```

ready래치는 작업자 스레드들이 준비과 완료됐을음 타이머 스레드에 통지할때 사용한다.

통지를 끝낸 작업자 스레드들은 두번째 래치인 start가 열리기를 기다린다. 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start.countDown을 호출하여 기다리던 작업자 스레드들을 깨운다. 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다. done래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다. 타이머 스레드는 done래치가 열리자마자 깨어나 종료 시각을 기록한다.

time메서드에 넘겨진 실행자는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메서드는 결코 끝나지 않는다.스레드 기아 교착상태

InterruptedException을 캐치한 작업자 스레드는 Thread.currentThread().interrupt()관용구를 사용해 인터럽트를 되살리고 자신은 run메서드에서 빠져나온다. 이렇게 해야 실행자가 인터럽트를 적절하게 처리할 수 있다.

시간 간격을 잴 떄는 항상 System.currentTimeMillis가 아닌 System.nanoTime을 사용하자.

어쩔수 없이 레거시코드 (wait,notify)를 사용해야 하는경우

```java
synchronized(obj){
  while(<조건이 충족되지 않았다.>){
    obj.wait();//락을 놓고 깨어나면 다시 잡는다.

  ...//조건이 충족됬을때 수행
  }
}
```

wait메서드를 사용할 때는 반드시 대기 반복문(wait loop)관용구를 사용하라 반복문 밖에서는 절대로 호출하지 말자.

조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황

-   스레드가 notify를 호출한 다음 대기중이던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.

-   조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 공개된 객체를 락ㅇ로 사용해 다기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이문제에 영향을 받는다.

-   깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.

-   대기 중인 스레드가(드물게) notify없이도 깨어나는 경우가 있다. 허위각성이라는 현상

이와 관련하여 notify와 notifyAll중 무엇을 선택하느냐 하는 문제도 있다. 일반적으로 notifyAll을 사용하라는게 합리적이고 안전한 조언

모든스레드가 같은 조건을 기다리고, 조건이 한번 충족될 떄마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll대신 notify를 사용해 최적화 할 수 있다

하지만 이상의 전제 조건들이 만족될지라도 notify대신 notifyAll을 사용해야 하는 이유는

외부로 공개된 객체에 대해 실수로 혹은 악의적으로 notify를 호출하는 상황에 대비하기 위해 wait를 반복문 안에서 호출했듯 notifyAll을 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait를 호출하는 공격으로부터 보호할 수 있다. 그런 스레드가 중요한 notify를 삼켜버린다면 꼭 깨어났어야 할 스레드들이 영원히 대기하게 될 수 있다.

---

wait와 notify를 직접 사용하는 것은 어셈블리 언어로 프로그래밍하는 것과 같다

코드를 새로 작성한다면 wait와 notify를 쓸 이유가 전혀 없다.

만약 쓴다면 wait는 항상 표준 관용구에 따라 while문안에 notify 대신 nofityAll을 사용하자

notify를 사용한다면 응답불가 상태에 빠지지 않게 주의

java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다. 바로 실행자 프레임워크, 동시성 컬렉션(concurrent collection), 동기화 장치(synchronizer)다
