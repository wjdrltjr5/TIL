## 지연 초기화는 신중히 사용하라

---

지연 초기화(lazy initializaion)는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 그래서 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다. 이기법은 정적필드와 인스턴스 필드 모두에 사용할 수 있다.

지연초기화는 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

다른 모드 최적화와 마찬가지로 지연 초기화에 대해 해줄 최선의 조언은 필요할 때까지는 하지마라.이다. 지연초기화는 양날의 검이다. 클래스 혹은 인스턴스를 생성 시의 초기화 비용은 줄지만 그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다. 그렇기에 실제로는 성능을 더 느려지게 할 수도 있다.

해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다. 알려면 결국 전후의 성능을 측정해봐야 한다.

멀티스레드 환경에서는 지연 초기화를 하기가 까다롭다. 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화 해야 한다.

대부분의 상황에서 일반적인 초기화가 지연 초기화 보다 낫다.

```java
//인스턴스 필드를 초기화하는 일반적인 방법
private final FieldType fiel = computeFieldValue();
```

지연 초기화가 초기화 순환성(initializaion circularity)을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용하자.

```java
//인스턴스 필드의 초기화 synchronized 접근자 방식

private FieldType field;

private synchfronized FieldType getField(){
  if(field == null){
    field = computeFieldValue();
  }
  return field;
}
```

성능때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스(lazy initialization holder class)관용구를 사용하자

```java
// 정적 필드용 지연 초기화 홀더 클래스 관용구
private static class FieldHolder{
  static final FieldType field = computeFieldValue();
}

private static FieldType getField(){return FieldHolder.field;}
```

getField가 호출된 순간 FieldHolder.field 가 처음 읽히면서 비로소 FieldHolder클래스 초기화를 촉발한다.

이 코드의 좋은점은 getField메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는 것.

일반적인 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화 할것이다. 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그 다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.

성능때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라. 초기화된 필드에 접근할 떄의 동기화 비용을 없애준다(item79)

한번은 동기화 없이 검사, 한번은 동기화 하여 검사한다. 두번째 검사에서도 필드가 초기화 되지 않았을 때만 초기화한다. 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatitle로 선언해야 한다.

volatile keyword는 Java 변수를 Main Memory에 저장하겠다라는 것을 명시하는 것입니다.
매번 변수의 값을 Read할 때마다 CPU cache에 저장된 값이 아닌 Main Memory에서 읽는 것입니다.
또한 변수의 값을 Write할 때마다 Main Memory에 까지 작성하는 것입니다. 즉 모든 스레드가 같은 값을 가지도록

```java
//인스턴스 필드 지연 초기화용 이중검사 관용구
private volatile FiedlType field;

private FieldType getField(){
  FieldType result = field;
  if(result != null){ // 첫 번째 검사(락 사용안함)
    return result;
  }
  synchronized(this){
    if(field == null){
      field = computeFieldValue();
    }
    return field;
  }
}
```

result지역변수는 필드가 이미 초기화된 상황(일반적인 상황)에서는 그 필드를 딱 한번만 읽도록 보장하는 역할을 한다. 반드시 필요하지는 않지만 성능을 높여주고 저수준 동시성 프로그래밍에 표준적으로 적용되는 좋은 방식이다.

이중검사에는 언급해둘 만한 변종이 두 가지 있다. 이따금 반복해서 최화해도 상관없는 인스턴스 필드를 지연 초기화 해야 할때가 있는데 이런 경우라면 이중검사에서 두 번째 검사를 생략할 수 있다. (단일검사 관용구가됨)

```java
//단일검사 관용구 초기화가 중복해서 일어날 수 있음
private volatile FieldType field;

private FieldType getField(){
  FieldType result = field;
  if(result == null){
    field = result = computeFieldValue();
  }
  return result
}
```

이번 아이템의 모든 초기화 기법은 기본 타입 필드와 객체 참조 필드 모두에 적용할 수 있다. 이중검사와 단일검사관용구를 수치 기본 타입필드에 적용한다면 필드의 값을 null대신 0과 비교

모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일검사의 필드 선언에서 volatile한정자를 없애도 된다.

---

대부분의 필드는 지연시키지 말고 곧바로 초기화 해야 한다.

성능 떄문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자.

인스턴스 필드엔ㄴ 이중검사 관용구를, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자. 반복해 최화해도 괜찮은 인스턴스 필드에는 단일검사 관용구도 고려 대상이다.
