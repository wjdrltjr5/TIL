## @Override 어노테이션을 일관되게 사용하라

---

자바가 기본적으로 제공하는 어노테이션 중 보통의 프로그래머에게 가장 중요한 것은 @Override일것이다. @Override는 메서드 선언에만 달 수 있다. 이 어노테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.

```java
//영어 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그를찾아보자
public class Bigram{
  private final char first;
  private final char second;

  public Bigram(char first, char second){
    this.first = first;
    this.second = second;
  }

  public boolean equals(Bigram b) {
     return b.first == first && b.second == second;
  }

  public int hashCode(){
    return 31 * first + second;
  }

  public static void main(String[] args){
    Set<Bigram> s = new HashSet<>();
    for(int i = 0; i < 10; i++){
      for(char ch = 'a'; ch <='z'; ch++){
        s.add(new Bigram(ch,ch));
      }
    }
    System.out.println(s.size());
  }
}
```

main 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램26개를 10번 반복해 집합에 추가한 다음, 그집합의 크기를 출력한다. Set은 중복을 허용하지 않으니 26이 출력될 거 같지만, 실제로는 260이 출력된다.

Biggram 작성자는 equals메서드를 재정의하려 한것으로 보이는데(item10) hashcode도 함께 재정의해야 한다는 사실을 잊지 않았다(item11) 그런데 equals를 오버라이딩 한것이 아니라 오버로딩을 해버렸다(item52)

Object.equals() 메서드는 객체와 다른 객체가 동일한 지 여부를 반환한다. equals를 오버라이딩 하지 않았을 경우 최상위 객체인 Object의 메서드가 호출된다. 이 경우 오직 자기 자신하고만 같다. (메모리 주소가 동일)

Object의 equals를 재정의 하려면 매개변수타입을 Object로 해야만 하는데 그렇게 하지않은것이다.

Object의 equals는 == 연산자와 똑같이 객체 식별성(identity)만을 확인한다. 따라서 같은 소문자를 소유한 바이그램 10개 각각이 서로 다른 객체로 인식되고, 결국 260을 출력한 것이다.

이오류는 컴파일러가 찾아낼 수 있지만, 그러려면 Object.equals를 재정의한다는 의도를 명시해야 한다.

```java

@Override
public boolean equals(Bigram b){
  return b.first == first && b.second == second;
}// 컴파일시 컴파일러가 오류 발생

//수정본
@Override
public boolean equals(Object o){
  if(!(o instanceof Bigram)){
    return false;
  }
  Bigram b = (Bigram)o;
  return b.first == first && b.second == second;
}
```

상위 클래스의 메서드를 재정의 하려는 모든 메서드에 @Override 어노테이션을 달자.(단 구체클래스에서 상위 클래스의 추상클래스 구현한거 빼고 달아도 되긴함)

---

재정의한 모든 메서드에 @Override 어노테이션을 의식적으로 달면 여러분이 실수했을때 컴파일러가 발로 알려줄 것이다. 예외는 한가지 뿐이다. 구체클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 이 어노테이션을 달지 않아도 된다.
