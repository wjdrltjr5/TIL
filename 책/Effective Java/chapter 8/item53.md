## 가변인수는 신중히 사용하라

---

```java
//간단한 가변인수
static int sum(int ...args){
  int sum = 0;
  for(int arg : args){
    sum += arg;
  }
  return sum;
}
```

```java
//잘못구현한 가변인수
static int sum(int ...args){
  if(args.length == 0){
    throw new IllgalArgumentException("인수가 1개 이상 필요합니다.");
  }
  int min = args[0];
  for(int i = 1; i < args.length; i++){
    if (args[i] < min){
        min = args[i];
    }
  return min;
  }
}
```

가장 심각한 문제는 인수를 0개만 넣어 호출하면 런타임에 실패한다는 점이다. 코드도 지저분하고 args유효성 검사를 명시적으로 해야하고 min의 초깃값을 Integer.MAX_VALUE로 설정하지 않고는 (더 명료한) for-each문도 사용할 수 없다.

```java
//인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법
static int min(int firstArg, int... remainingArgs){
  int min = firstArg;
  for(int arg : remainingArgs){
    if(arg < min){
      min = arg;
    }
    return min;
  }
}
```

가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다 printf는 가변인수와 한 묶음으로 자바에 도입되었고, 이때 핵심 리플렉션기능(item65)도 재정비 되었다. printf와 리플렉션 모두 가변인수 덕을 톡톡히 보고있다.

성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다. 가변인수 메서드는 호출될 때마다 배열을 새로 하나 만들고 할당하고 초기화한다.

다행이 이 비용을 감당할 수는 없지만 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴이 있다. 예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 해보자.

예로 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 했을때 인수가0개인것부터 4개인 것까지, 총 5개를 다중정의하자.

```java
public void test(){}
public void test(int a){}
public void test(int a, int b){}
public void test(int a, int b, int c){}
public void test(int a, int b, int c, int... d){}
```

이런식이면 메서드 호출중 5%만 배열을 생성한다 보통떄는 별 이득이 없지만 특수한 상황에서는 이득이다.

EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 한다.
EnumSet은 비트필드(item36)를 대체하면서 성능까지 유지해야 하므로 아주 적절하게 활용한 예이다.

---

인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다. 메서드를 정의할 떄 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.
