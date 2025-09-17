## 불필요한 객체 생성을 피해라

```java
//따라 하지 말것!
String a = new String("bikini");
```

필요한 경우가 아니라면 박싱된 기본 타입보다는 그냥 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

비싼 객체는 되도록 재사용하자

---

## 다 쓴 객체 참조를 해제하라

-   메모리 누수가 일어나는 코드

```java
public class Stack{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITAL_CAPACITY = 16;

  public Stack(){
    element = new Object[DEFAULT_INITAL_CAPACITY];
  }
  public void push(Object e){
    ensureCapacity();
    elements[size++] = e;
  }
  public Object pop(){
    if(size == 0){
      throw new EmptyStackException();
    }
    return elements[--size];
  }
  /**
    원소를 위한 공간을 적어도 하나 이상 확보한다.
    배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
  */
  private void ensureCapacity(){
    if(elements.length == size){
      elements = Arrays.copyOf(elements, 2 * size+1);
    }
  }
}
```

위 코드에서는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 이스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.
(배열의 활성영역 밖의 참조들이 모두 해당)
가비지 컬렉션 언어에서는 메모리 누수가 찾기 어렵다.

객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수하지 못함

### 해결방법

```java
public Object pop(){
  if(size == 0){
    throw new EmptyStackException();
  }
  Object result = elements[--size]; //단지 index 번호를 낮추는것 뿐 아니라 해당 인덱스를 비우기 !
  elements[size] = null;
  return result;
}
```

즉 자기 메모리를 직접 관리하는 클래스는 메모리 누수에 주의해야함
