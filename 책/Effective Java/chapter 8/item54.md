## null이 아닌, 빈 컬렉션이나 배열을 반환하라.

---

```java
private final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈목록을 반환
 * 단 재고가 없다면 null을 반환.
 *
*/
public List<Cheese> getCheese(){
  return cheeseInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock);
}
```

사실 재조가 없다고 해서 특별히 취급할 이유는 없다. 그럼에도 이 코드처럼 null을 반환한다면, 클라이언트는 이 null상황을 처리하는 코드를 추가로 작성해야 한다.

```java
List<Cheese> cheeses = shop.getCheese();
if(cheeses != null && cheeses.contains(Cheese.STILTON))
  System.out.println("좋았어, 바로 그거야 ");
```

컬렉션이나 배열같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할 때면 항시 이와 같은 방어 코드를 넣어줘야 한다. 클라이언트에서 방어 코드를 빼먹으면 오류가 발생할 수 있다. (실제로 객체가 0일 가능성이 거의 없을경우 한참뒤에 오류가 발생할 수 있다.)

null을 반환하려면 반환하는 쪽에서도 상황을 특별히 취급해줘야 해서 코드가 더 복잡해 진다.

빈컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장이 있는데 두가지 면에서 틀린 주장

-   성능분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는한(item67)이정도의 성능차이는 신경 쓸 수준이 못 된다.
-   빈 컬렉션과 배열은 새로 할당하지 않고도 반환할 수 있다.

```java
//빈컬렉션을 반환하는 올바른 예
public List<Cheese> getCheese(){
  return new ArrayList<>(cheeseInStock);
  }
```

가능성은 작지만 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수도 있다. 해법은 매번 똑같은 빈 불변 컬렉션을반환하는 것이다.

불변객체는 자유롭게 공유해도 안전하다(item17) 집합이 필요하면 Collections.emptySet을 맵이 필요하면 Collections.emptyMap을 사용하면 된다.

```java
//이역시 최적화에 해당되니 성능비교를 해보고 필요하면 사용
public List<Cheese> getCheeses(){
  return cheesesInStock.isEmpty() ? Collection.emtpyList()
            : new ArrayList<>(CheeseInStock);
}
```

배열을 쓸때도 마찬가지다. 절때 null을 반환하지 말고 길이가 0인 배열을 반환하라.

```java
//길이가 0일수도 있는 배열을 반환하는 올바른 방법
public Cheese[] getCheese(){
  return cheeseInStock.toArray(new Cheese[0]);
}
```

이 방식이 성능을 떨어 트릴것 같다면 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다 길이 0인 배열은 모두 불변이기 때문이다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheese(){
  return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

최적화 버전에 getCheese는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray를 호출한다

```java
//나쁜예 배열을 미리 할당하면 성능이 나빠진다.
return cheeseInStock.toArray(new Cheese[cheeseInStock.size()]);
```

---

null이 아닌, 빈 배열이나 컬렉션을 반환하라. null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.
