## 명명 패턴보다 애너테이션을 사용하라

---

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명패턴을 적용해왔다. (Junit은 버전3까지 테스트 메서드 이름을 test로 시작하게끔 했다. 최근에는 @Test어노테이션으로 사용) 이방법은 효과적이지만 단점도 크다(오타가 나면 안됨, 올바른 프로그램 요소에서만 사용되리라 보증할 수 없음,프로그램 요소를 매개변수로 전달할 마땅한 방법이 이 없음) 이러한 단점을 해결하기 위해 어노테이션을 사용

```java
//마커(maker)애너테이션 타입 선언
//아무 매개변수 없이 단수히 대상에 마킹한다는 뜻.
@Retention(RetentionPolicy.RUNTIME)//런타임에도 유지
@Target(ElementType.METHOD)// 메서드 선언에서만 사용해야함
public @interface Test{}
```

```java
public class Sample{
 @Test
 public static void m1(){}//성공해야한다.

 public static void m2(){}

 @Test
 public static void m3(){//실패해야한다.
  throw new RuntimeException("실패");
 }

 public static void m4(){}

 @Test
 public void m5(){}//잘못사용한예 정적 메서드가 아님.

 public static void m6(){}

 @Test
 public static void m7(){
    throw new RuntimeException("실패");
 }
public static void m8(){}


}

```

Test 어노테이션이 Sample클래스의 의미에 영향을 주지는 않고 추가 정보를 제공할 뿐이다. 이 어노테이션에 관심있는 도구에서 특별한 처리를 할 기회를 준다.

```java
//마커 어노테이션을 처리하는 프로그램
import java.lang.reflect.*;

public class RunTests{
  public static void main(String[] args)throws Exception{
    int tests = 0;
    int passed = 0;

    Class<?> testClass = Class.forName(args[0]);
    for(Method m : testClass.getDeclareMethods()){
      //isAnnotationPresent 실행할 메소드를 찾아주는 메서드
      if(m.isAnnotationPresent(Test.class)){
        tests++;
        try{
          m.invoke(null);
          passed++;
        }catch(InvocationTargetException wrappedExc){
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + "실패 : " + exc);
        }catch(Exception exc){
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
    System.out.printf("성공 : %d, 실패 : %d%n", passed, tests - passed);
  }
}

```

RunTests는 정규화된 클래스 이름을 받아. @Test어노테이션이 달린 메서드를 차례로 호출한다.테스트 메서드가 예외를 던지면 리플렉션 매커니즘이InvocationTargetException으로 감싸서 다시 던진다. 그래서 이 프로그램은 InvocationTargetException을 잡아 원래 예외에 담긴 실패 정보를 getCause()추출해 출력한다.

```java
//배열 매개변수를 받는 어노테이션 타입
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
  Class<? extends Throwable>[] value();
}
```

자바8에서는 여러 개의 값을 받는 애너테이션을 배열 매개변수를 사용하는 대신 애너테이션에 @Repeatable 메타 애너테이션을 다는 방식으로 만들 수 있다. @Repeatable을 단 어노테이션은 하나의 프로그램 요소에 여러 번 달 수 있다. 주의할점이 있는데 @Repeatable을 단 어노테이션을 반환하는 컨테이너 어노테이션을 하나 더 정의하고 @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다, 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다 마지막으로 컨테이너 어노테이션 타입에는 적절한 보존정책(@Retention)과 적용대상(@Target)을 명시해야 한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest{
    Class<? extends Throwable> value();
}
//컨테이너 어노테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer{
  ExceptionTest[] value();
}

```

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad(){...}
```

반복 가능 어노테이션을 여러개 달면 하나만 달았을 때와 구분하기 위해 해당 컨테이너 어노테이션 타입이 적용된다. getAnnotationsByType메서드는 이 둘을 구분하지 않아서 반복 가능 어노테이션과 그 컨테이너 어노테이션을 모두 가져오지만, isAnnotationPresent 메서드는 둘을 명확히 구분한다. 따라서 반복 가능 어노테이션을 여러번 단 다음 isAnnotationPresent로 반복 가능 어노테이션이 달렸는지 검사한다면 "그렇지 않다" 라고 알려준다(컨테이너가 다르기 때문) 그결과 어노테이션을 한 번만 단 메서드를 무시하고 지나친다. 그래서 달려 있는 수와 상관없이 모두 검사하려면 이 둘을 따로따로 확인해야 한다.

```java
if (m.isAnnotationPresent(ExceptionTest.class)
    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
  tests++;
  try {
    m.invoke(null);
    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
    Throwable exc = wrappedExc.getCause();
    int oldPassed = passed;
    ExceptionTest[] excTests =
      m.getAnnotationsByType(ExceptionTest.class);
    for (ExceptionTest excTest : excTests) {
      if (excTest.value().isInstance(exc)) {
        passed++;
        break;
      }
    }
    if (passed == oldPassed)
      System.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
}
```

반복 가능 어노테이션을 사용해 코드 가독성을 높일 수 있지만 애너테이션을 선언하고 이를 처리하는 부분에서는 코드 양이 늘어나며, 처리코드가 복잡해 오류가 날 가능성이 커진다.

---

어노테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다. 자바 프로그래라면 예외 없이 자바가 제공하는 어노테이션 타입들을 사용해야 한다.(item40,27)
