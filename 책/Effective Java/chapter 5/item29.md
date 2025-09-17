## 이왕이면 제네릭 타입으로 만들라

---

```java
//Object 기반 스택-제네릭이 절실한 강력 후보
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
  		Object result = elements[--size];
  		element[size] = null;
        return result;
    }

  	public boolean isEmpty() {
  		return size = 0;
  	}

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

```java
//제네릭
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
```

다만 위 코드는 오류나 경고가 발생함.

```java
제네릭은 실체화 불가 타입으로 배열을 만들 수 없기 때문!
Stack.java:8: generic array creation
  		elements = new E[DEFAULT_INITIAL_CAPACITY];

```

위 오류의 해결방안

1. 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법.

<br>Object배열을 생성후 제네릭 배열로 형변환 해보자

```java
Stack.java:8: warning: [unchecked] unchecked cast
  		elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY];
```

그럼 위 경고를 내보낸다 이렇게도 할 수 있지만 일반적으로 타입 안전하지 않다. 컴파일러는 이 프러그램이 타입 안전한지 증명할 방법이 없지만 우리는 할 수 있다. 따라서 이 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 우리 스스로 확인해야 한다. <br>

문제의 배열 element는 private필드에 저장되고, 클라이언트에 반환되거나 다른 메서드에 전달되는 일이 전혀 없다. push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E다. 따라서 이 비검사 형변환은 확실히 안전하다. 비검사 형변환이 안전함을 직적 증명했다면 범위를 최소로 좁혀 @SuppressWarnings 어노테이션으로 해당 경고를 숨긴다.

```java
@SuppressWarning("unchecked")
public Stack(){
  elements = (E[])new Object[DEFAULT_INITIAL_CAPACITY];
}
```

2. element 필드의 타입을 E[] 에서 Object[]로 바꾸는 방법.
   <br>
   이렇게 하면 첫 번째와는 다른 오류가 발생

```java
Stack.java:19: incompatible types
found: Object, required: E
  E result = element[--size];
```

배열이 반환한 요소를 E로 형변환 하면 오류 대신 경고가 발생함

```java
Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
  E result = (E)element[--size];
```

E는 실체화 불가 타입이므로 컴파일러는 런타임에 이루어지는 형변환이 안전한지 증명할 방법이 없다. 마찬가지로 우리가 직접 증명하고 경고를 숨길 수 있다.

```java
public E pop() {
        	if (size == 0)
            	throw new EmptyStackException();

        // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        //경고 숨기는 범위 최소화
        @SuppressWarnings("unchecked") E result = (E) elements[--size];

        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
```

현업에서는 첫번째 방식을 더 선호(가독성 좋음)자주사용 하지만 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염을 일으킨다. 힙오염이 맘에 걸리는 프로그래머는 두 번째 방식을 고수하기도 한다.

---

클라이언트에서 직접 형변환 해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라. 그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다. 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.
