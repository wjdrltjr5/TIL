## 상속보다는 컴포지션을 사용해라

---

### 문제점

메소드 호출과 달리 상속은 캡슐화를 깨트린다.

-   상위클래스가 어떻게 구현되느냐에 따라 하위 클래서의 동작에 이상이 생길 수 있다.
-   상위클래스는 릴리스마다 내부 구현이 달라질 수 있다.(하위클래스가 오작동할 수 있다.)

잘못된 예

```java
public class InstrumentedHashSet<E> extends HashSet {
     // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

}

```

-   HashSet의 addAll 메서드는 add 메서드를 사용해서 구현되어 있다.
-   addAll은 addCount를 더한 후, HashSet의 addAll을 호출한다.
-   HashSet의 addAll은 각 원소를 add 메서드를 호출해 추가하는데, 이때 불리는 add는 재정의한 메서드다. 따라서 addCount의 값이 중복으로 더해지게 된다.
-   이 경우 하위 클래스에서 addAll 메서드를 재정의하지 않으면 문제를 고칠 수 있다. 하지만 HashSet의 addAll이 add 메서드를 이용해 구현했음을 가정한 해법이라는 한계를 가진다.
-   이처럼 자신의 다른 부분을 사용하는 '자기 사용(self-use)' 여부는 해당 클래스의 내부 구현 방식에 해당하며, 다음 릴리즈에서도 유지될지는 알 수 없다. 따라서 이런 가정에 기댄 InstrumentedHashSetUseExtends도 깨지기 쉽다.

addAll 메서드를 다른 식으로 재정의할 수도 있다. 하지만 상위 클래스의 메서드 동작을 다시 구현하는 것은 어렵고, 시간도 더 들고, 오류를 내거나 성능을 떨어뜨릴 수도 있다. 또한 하위 클래스에서는 접근할 수 없는 private 필드를 써야 하는 상황이라면 이 방식으로는 구현자체가 불가능하다.

다음 릴리스에서 상위 클래스에 새로운 메서드를 추가한다면?(만약 이상이 없다 하더라도 추가된 메소드가 하위클래스에 추가된 메서드와 시그니처가 같고 반환타입이 다르다면? 바로 컴파일 오류)

---

### 해결방법

기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.(기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 이러한 설계를 컴포지션(composition)이라 한다.)

새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 전달(forwarding)이라 하며, 새 클래스의 메서드들을 전달 메서드(forwarding method)라 부른다.
그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

```java
// 래퍼 클래스 상속대신 컴포지션을 사용했다.
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

```java
//재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public void clear() {
        s.clear();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public boolean isEmpty() {
        return s.isEmpty();
    }

    public int size() {
        return s.size();
    }

    public Iterator<E> iterator() {
        return s.iterator();
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean remove(Object o) {
        return s.remove(o);
    }

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}
```

이제 InstrumentedSet은 HashSet의 모든 기능을 정의한 Set인터페이스를 활용해 설계되어 견고하고 아주 융ㄴ하다. 구체적으로는 Set인터페이스를 구현했고,Set의 인스턴스를 인수로 받는 생성자를 하나 제공한다.임의의 Set에 계측 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다.

상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다. 하지만 컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

-   다른 Set 인스턴스를 감싸고(wrap) 있다는 뜻에서 래퍼 클래스라고 한다.
-   다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴(Decorator pattern)이라고 부른다. 단, 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.
-   래퍼 클래스는 단점이 거의 없다. 래퍼 클래스가 콜백(callback) 프레임워크와는 어울리지 않는다는 점만 주의하도록 하자.

---

상속은 강력하지만 캡슐화를 해친다는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때도 안심할 수만은 없는게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사요야하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.
