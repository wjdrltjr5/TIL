## 전통적인 for문보다 for-each문을 사용하라

---

item45에도 이야기 했듯, 스트림이 제격인 작업이 있고 반복이 제격인 작업이 있다.

```java
// 전통적인 for문으로 컬레션 순회.
for(Iterator<Element> i = c.iterator(); i.hasNext();){
  Element e = i.next();
}

//전통적인 for문으로 배열 순회
for(int i = 0; i< a.length(); i++){
  ...
}
```

while문 보다는 낫지만(item57) 가장 좋은 방법은 아니다. 코드를 지저분하게 할 뿐 우리에게 진짜 필요한 건 원소들 뿐이다 더구나 이처럼 쓰이는 요소가 늘어나면 오류가 생길 가능성이 높아진다. 1회 반복에서 반복자는 세번 등장하며, 인덱스는 네번이나 등장하여 변수를 잘못 사용할 틈새가 넓혀진다. 혹시나 잘못된 변수 사용시 컴파일러가 잡아주리라는 보장도 없다. 컬렉션이냐 배열이냐에 따라 코드 형태가 달라지므로 주의해야 한다.

for-each를 사용하면 모두 해결된다. 반복자와 인덱스변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일도 없다. 하나의 관용구로 컬렉션과 배열 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다.

```java
for(Element e : elements){}
```

컬렉션을 중첩해 순회해야 한다면 for-each문의 이점은 더욱 커진다.

```java
//버그가 있는 코드
enum Suit {CLUB, DIAMOND, HEART, SPADE}
enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE
  , TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Coolection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();){
  for(Iterator<Rank> j = ranks.iterator(); j.hasNext();){
    deck.add(new Card(i.next(), j.next()));
  }
}
```

버그를 찾기 어렵지만 문제는 바깥 컬렉션(suits)반복자에서 next메서드가 너무 많이 불린다는 것 마지막줄의 i.next()를 주목하자 이 next()는 숫자(Suit) 하나당 한번씩만 불려야하는데 안쪽 반복문에서 호출하는 바람에 카드(Rank)하나당 한 번씩 불리고 있다. 그래서 숫자가 바닥나면 반복문에 NoSuchElementException을 던진다. 정말 운이 나빠서 바깥 컬렉션의 크기가 안쪽 컬렉션의 크기의 배수라면 예외를 던지지 않고 종료한다.

```java
//주사위를 두번 굴렸을때 나오는 경우의 수 같은 버그 다른증상
enum Face{ ONE, TWO, THREE, FOUR, FIVE, SIX}
...
Collection<Face> faces = EnumSet.allOf(Face,class);

for(Iterator<Face> i = faces.iterator(); i.hasNext();){
  for(Iterator<Face> j = faces.iterator(); j.hasNext();){
    System.out.println(i.next() + " " + j.next());
  }
}
```

예외를 던지지는 않지만 가능한 조합을 ONE ONE 부터 SIX SIX 까지만 던지고 끝난다.(총 36개가 나와야함)

```java
//해결방법 가독성이 좋진 않음
enum Face{ ONE, TWO, THREE, FOUR, FIVE, SIX}
...
Collection<Face> faces = EnumSet.allOf(Face,class);

for(Iterator<Face> i = faces.iterator(); i.hasNext();){
  Suit suit = i.next();
  for(Iterator<Face> j = faces.iterator(); j.hasNext();){
    System.out.println(suit + " " + j.next());
  }
}
```

```java
//for-each문을 사용하여 해결
for(Suit suit : sutis){
  for(Rank rank : ranks){
    deck.add(new Card(suit, rank));
  }
}
```

for-each를 사용할 수 없는 상황이 세가지 있다.

-   파괴적인 필터링(destructive filtering) - 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다. 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

-   변형(transforming) 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.

-   병렬 반복(parallel iteraion) 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

세가지 상황 중 하나에 속할 때는 일반적인 for문을 사용하되 이번 아이템에서 언급한 문제들을 경계하기 바란다.

for-each문은 컬렉션,배열, iterable인터페이스를 구현한 객체라면 순회할 수 있다.

원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 고민해보자.

---

전통적인 for과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다. 성능저하도 없다. 가능한 모든 곳에서 for문대신 for-each를 사용하자.

for-each는 배열을 복사해서 가져오기 때문에 값을 변경할 수 는 없다.
