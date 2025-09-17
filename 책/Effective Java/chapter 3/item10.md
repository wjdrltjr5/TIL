## equals는 일반 규약을 지켜 재정의 하라.

---

equals 메서드는 동치 관계를 구현하며 다음을 만족한다.

-   반사성 : null이 아닌 모든 참조 값 x에 대해 ,x.equals(x)는 true이다.
-   대칭성 : null이 아닌 모든 참조 값 x,y에대해 x.equals(y)가 true이면 y.equals(x)도 true이다.
-   추이성 : null이 아닌 모든 참조 값 x,y,z 에대해 x.equals(y)가 트루이고, y.equals(z) 트루이면 ,x.equals(z)도 true이다.
-   일관성 : 반복해서 호출시 항상 같은 값을 반환해야 한다.
-   null-아님 : null이 아닌 참조 값 x에 대해, x.equals(null)은 false이다.

꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 원하는 비교를 정확히 수행해준다. 만약 재정의 한다면 해당 클래스의 핵심 필드가 5가지 규약을 확실히 지켜가며 비교해야 한다.

item 11
equals를 재정의 하려거든 hashCode도 재정의 하라 equals가 true이면 hashCode도 같은 값을 반환해야한다.
