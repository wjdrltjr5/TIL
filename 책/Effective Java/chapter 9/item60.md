## 정확한 답이 필요하다면 float와 double은 피하라

---

float와 double타입은 과학과 공학 계산용으로 설계되었다. 이진 부동소수점연사에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었다. 따라서 정확한 결과가 필요할 때는 사용하면 안된다. float와 double타입은 특히 금융관련 계산과는 맞지 않는다. 0.1혹은 10의 음의 거듭제곱수를 표현할 수 없기 때문이다.

ex. 주머니에 1.03달러가 있었는데 그중 42센트를 썻다고 했을때 남은돈은?

```java
//어설프게 작성한 코드
System.out.println(1.03 - 0.42);
```

이코드는 0.610000000000001을 출력한다.

주머니에 1달러가 있었는데 10센트짜리 사탕 9개를 샀을때 남은온든?

```java
System.out.println(1.00 - 9 * 0.10);
```

이코드는 0.0999999999998을 출력한다.

결과값을 반올림하면 해결되리라 생각할지 모르지만, 반올림을 해도 틀린 답이 나올 수 있다. 예로 1달러가 있고 선반에 10센트,20센트,30센트 ... 1달러 짜리의 맛있는 사탕이 있을때 10센트부터 하나씩 살 수 있을 때까지 사보자. 사탕을 몇 개나 살 수 있고, 잔돈은 얼마나 남을까?

```java
//오류가 발생하는 코드 금융계산에 부동소수 타입사용
public static void main(String[] args){
  double funds = 1.00
  int itemsBougth = 0;
  for(double price = 0.10; funds >= price; price += 0.10){
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemBought + "개 구입");
  System.out.println("잔돈(달러): " + funds);
}
```

프로그램 실행시 사탕3개를 구입후 잔돈은 0.3999999999999달러가 남았음을 알게된다. 물론 잘못된 결과.

이문제를 올바로 해결하려면 금융게산에는 BigDecimal(실수표현), int 혹은 long을 사용해야 한다.

```java
//double 대신 BigDecimal사용
public static void main(String[] args){
  final BigDecimal TEN_CENTS = new BigDecimal(".10");

  int itemsBougth = 0;
  BigDecimal funds = new BigDecimal("1.00");
  for(BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0;
       price = price.add(TEN_CENTS)){
    funds = funds.subtract(price);
    itemsBought++;
  }
  System.out.println(itemBought + "개 구입");
  System.out.println("잔돈(달러): " + funds);
}
```

이프로그램을 사용하면 사탕4개를 구입후 잔돈은 0달러가 남는다.

하지만 BigDecimal에도 단점이 두가지 있다. 기본타입보다 쓰기가 불편하고 훨씬 느리다. 단벌성 계산이라면 느리다는 문제는 무시할 수 있지만, 쓰기 불편하다는 점은 아쉽다.

BigDecimal의 대안으로 int 혹은 long타입을 쓸 수 있다. 그럴경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야한다.

```java
//정수타입을 사용한 해법
public static void main(Stringp[] args){
  int itemsBought = 0;
  int funds = 100;

  for(int price = 10; funds >= price; price += 10){
    funds -= price;
    itemsBougnt++;
  }
  System.out.println(itemsBought + "개 구입 ");
  System.out.println("잔돈(센트): " + funds);
}
```

---

정확한 답이 필요한 계산에는 float, double을 피하라. 소수점 추적은 시스템에 맡기고 코딩 시의 불편함이나 성능저하를 신경 쓰지 않겠다면 BigDecimal을 사용하라. BigDecimal이 제공하는 여덟가지 반올림 모드를 이용하여 계산에서 아주 편리한 기능이다. 반면 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크기 않다면 int나 long을 사용하라. 숫자를 아홉 자리 십진수로 표현할 수 있다면 int를 사용하고, 열여덟자리 십진수로 표현할 수 있다면 long을 사용하라. 열여덟자리를 넘기면 BigDecimal을 사용해야 한다.
