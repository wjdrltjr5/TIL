## public 클래스에서는 public 필드가 아닌 접근자 메소드를 사용하라

---

-   잘못된 예

```java
class Point{

  public double x;
  public double y
}
```

-   올바른 예

```java
class Point{
  private double x;
  private double y;

  public double getX(){return x;}
  public double gety(){return y;}

  public void setX(double x){this.x = x;}
  public void setY(double y){this.y = y;}
}
```

public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면(final) 노출해도 덜 위험하지만 완전히 안심할 수 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변,가변 둘다) 필드를 노출하는 편이 나을 때도 있다.
