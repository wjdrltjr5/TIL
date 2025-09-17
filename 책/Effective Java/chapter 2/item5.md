## 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

---

클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이자원들을 클래스가 직접 만들게 해서는 안된다(의존주입할것)

-   정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.

```java
public class SpellChecker{
  private static final Lexicon dictionary = ...;

  private SpellChecker(){}

  public static boolean isValid(String word){...}
  public static List<String> suggestions(String type){...}
}
```

-   싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.

```java
public class SpellChecker{
  private final Lexicon dictionary = ...;

  private SpellChecker(...){}
  public static SpellChecker INSTANCE = new SpellChecker(...);

  public boolean isValid(String word){...}
  public List<String> suggestions(String typo){...}
}
```

두방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서는 그리 훌룡해 보이지 않는다.
사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

-   의존 객체 주입은 유연성과 테스트 용이성을 높여준다.

```java
public class SpellChecker{
  private final Lexicon dictionary;

  public SpellChecker(Lexicon dictionary){
    this.dictionary = dictionary;
  }
  public boolean isValid(String word){...}
  public static List<String> suggestions(String type){...}
}
```
