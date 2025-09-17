## 한정적 와일드카드를 사용해 API 유연성을 높이라.

---

앞장에서 말했단 매개변수화 타입은 불공변이다. 즉 서로다른 타입 List\<Type1>,List\<Type2>가 있을때 List\<Type1>은 List\<Type2>의 하위타입도 상위타입도 아니다.(ex. List\<String>, List\<Object>)
<br>
때론 불공변 방식보다 유연한 무언가가 필요하다.

```java
public class Stack<E>{
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
//와일드 카드 타입을 사용하지 않은 pushAll메서드 - 결함이 있음
public void pushAll(Iterable<E> src){
  for (E e : src){
    push(e);
  }
}
```

이메서드는 컴파일 되지만 완벽하지 않다. Iterable sec의 원소타입이 스택의 원소 타입과 일치하면 잘 작동하지만 Stack\<Number>로 선언한 후 pushAll(intVal)을 호출하면(intVal은 Integer타입) Integer는 Number의 하위타입 <br>

```java
//실행시 오류가 발생한다.
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers)
// 매개변수화 타입이 불공변이기 때문
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
              numberStack.pushAll(integers);
```

---

해결책 : 자바에서 지원하는 한정적 와일드 카드타입을 사용한다.

```java
public void pushAll(Iterable<? extends E> src){
  for (E e : src){
    push(e);
  }
}
```

```java
// 와일드 카드 타입을 사용하지 않은 popAll메서드
// 오류 발생
public void popAll(Collection<E> dst){
  while(!isEmpty()){
    dst.add(pop())
  }
}
//와일드카드 타입 적용
public void popAll(Collection<? super E> dst){
  while(!isEmpty()){
    dst.add(pop());
  }
}

```

<Strong>유연성을 극대화 하려면 원소의 생산자나 소비자용 입력매개변수에 와일드 카드 타입을 사용하라</Strong>
<br>
한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드 카드 타입을 써도 좋을게 없다.
<br>
펙스(PECS) : producer-extends,consumer-super<br>
매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고 소비자라면<? super T>를 사용하라. Stack의 예로 pushAll의 src매개변수는 Stack이 사용할 E 인스턴스를 생성함으로 src의 적절한 타입은 Iterable<? extends E>이다. popAll의 dst매개변수는 Stack으로부터 E인스턴스를 소비하므로 dst의 적절한 타입은 Collection<? super E>이다. PECS 공식은 와일드 카드 타입을 사용하는 기본 원칙이다.

---

클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다.

```java
//1번 방식 비한정적 타입 매배견수
public static <E> void swap(List<E> List, int i, int j);
//2번 방식 비한정적 와일드카드
public static void swap(List<?> list, int i, int j);
```

public API라면 두번째 방식이 좋다.<br>
메서드 선언에 타입 매개변수가 한 번만 나오면 와일드 카드로 대체하라. 이때 비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 변경

단 두번째 swap선언에는 문제가 하나 있는데 직관저긍로 구현한 코드는 컴파일 되지 않는다.

```java
public static void swap(List<?> list, int i, int j){
  list.set(i, list.set(j, list.get(i)));
}

오류메시지
Swap.java:5: error: incompatible types: Object cannot be converted to CAP#1
  list.set(i, list.set(j, list.get(i)));

where CAP#1 is a fresh type-variable:
 CAP#1 extends Object from capture of ?

```

원인은 리스트 타입이 List<?>인데, List<?>에는 null외에는 어떤 값도 넣을 수 없다는 데 있다. 해결방안은 와일드 카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드로 따로 작성하여 활용하는 방법

```java
public static void swap(List<?> list, int i, int j){
  swapHelper(list, i, j);
}
//와일드카드 타입을 실제 타입으로 바꿔주는 private도우미
private static <E> void swapHelper(List<E> list, int i, int j){
  list.set(i, list.set(j, list.get(i)));
}
```

swapHelper 메서드는 List\<E>임을 알고 있다. 즉 이 리스트에서 꺼내는 타입은 항상 E이고 E타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다.

---

조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야한다. PECS공식을 기억하자. 즉 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다. Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자.
