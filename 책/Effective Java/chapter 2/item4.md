## 인스턴스화를 막으려거든 private 생성자를 사용하라

---

단순히 정적 메서드와 적정 필드만을 담은 클래스를 만들고 싶을때
추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.

```java
public class UtilityClass{
  private UtilityClass(){
      throw new AssertionError();//혹시라도 클래스 내부에서 생성 방지
  }
  ... //생략
}
```
