## 익명 클래스보다는 람다를 사용하라

---

```java
//익명 클래스의 인스턴스를 함수 객체로 사용
Collections.sort(words, new Comparator<String>(){
  public int compare(String s1, String s2){
    return Integer.compare(s1.length(), s2.length());
  }
});
```

전략패턴처럼 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했다. 이코드에서는 Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다. 하지만 익명 클래스 방식은 코드가 너무 길기 때문에 함수형 프로그래밍에 적합하지 않다.

```java
//람다식 사용
Collections.sort(words, (s1,s2) -> Integer.compare(s1.length(), s2.length()));
```

람다,매개변수(s1,s2), 반환값의 타입는 각각 Comparator\<String>,String,int 컴파일러가 문맥을 살펴 타입추론해준다.(상황에 따라 컴파일러가 타입을 결정하지 못할 수도 있는데, 프로그래머가 직접 명시해줘야 한다.)타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.

람다 자리에 비교자 생성 메서드를 사용하면 이 코드를 더 간결하게 만들 수 있다. (item14,43)

```java
Collections.sort(words, comparingInt(String::length));
```

더나아가 자바 8때 List 인터페이스에 추가된 sort 메서드를 이용하면 더욱 짧아진다.

```java
words.sort(comparingInt(String::length));
```

과거 코드를 통한 예제

```java
public enum Operation(){
//   PLUS("+"){
//     public double apply(double x, double y){return x+y;}
//   }
  PLUS("+",(x,y) -> return x+y);

  private final String symbol;

  Operation(String symbol,DoubleBinaryOperator op){
    this.symbol = symbol;
    this.op = op;
  }

  @Override public String toString(){return symbol;}

  public double apply(double x, double y){
    return op.applyAsDouble(x,y);
  }
}
```

람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.(한줄일때 가장 좋고 길어야 세 줄 안에 끝내는 게 좋다. 가독성)

람다는 자신을 참조할 수 없다 람다에서의 this는 바깥 인스턴스를 가리킨다. 반면 익명 클래스의 this는 익명클래스 자신을 가리킨다. 그래서 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.

람다도 익명 클래스처럼 직렬화 형태가 구현별로(or 가상머신) 다를 수 있다. 그렇기에 람다를 직렬화 하는 일은 극히 삼가야 한다.(익명클래스도 마찬가지)

---

익명 클래스는(함수형 인터페이스가 아닌) 타입의 인스턴스를 만들때만 사용하라.
