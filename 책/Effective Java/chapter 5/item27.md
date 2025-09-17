## 비검사 경고(unchecked warnings)를 제거하라

---

대부분의 비검사 경고는 쉽게 제거할 수 있다. 코드를 다음처럼 잘못 작성했다고 해보자

```java
//컴파일러가 오류를 알려줌
Set<Lark> exaltation = new HashSet();

자바7부터는 다이어몬드 연산자만 사용해도 컴파일러가가 올바른
실제 타입 매개변수를 추론해줌

Set<Lark> exaltation = new HashSet<>();
```

경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("uncheked") 어노테이션을 달아 경고를 숨기자. 단, 타입안전함을 검증하지 않은 채 경고를 숨기면 안된다.(그 코드는 경고 없이 컴파일되겠지만, 런타임에는 여전히 ClassCastException을 던질 수 있다.) 반대로 안전하다고 검증된 비검사 경고를 숨기지 않고 그대로 두면 제거하지 않은 수많은 거짓 경고 속에 새로운 경고가 파묻힐 수 있다.<br>

@SuppressWarnings 어노테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있으며 항상 가능한 좁은 범위에 적용해야 한다.(변수선언, 아주 짧은 메서드 혹은 생성자 절대로 클래스 전체에 적용하지 말것!)

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size)
		return (T[]) Arrays.copyOf(elements, size, a.getClassO);
	System.arraycopy(elements, 0, a, 0, size);
	if (a.length > size)
		a[size] = null;
	return a;
}
/* 경고 발생
 ArrayList.java:305: warning: [unchecked] unchecked cast
	return (T[]) Arrays. copyOf (elements, size, a.getClassO);
  required: T[]
  found: Object []
 * /
```

어노테이션은 선언문에만 달 수 있기 떄문에 return문에서는 담는게 불가능 메서드전체에 다는건 범위가 넚어지니 어노테이션을 적용한 지역변수를 하나 생성후 반환값을 반환하자.

```java
public <T> T[] toArray(T[] a) {
	if (a.length < size) {
		// 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
		// 올바른 형변환이다.
		@SuppressWarnings("unchecked") T[] result =
			(T[]) Arrays.copyOf(elements, size, a.getClass());
		return result;
	}
	System.arraycopy(elements, 0, a, 0, size);
	if (a.length > size)
		a[size] = null;
	return a;
  }
```

@SuppressWarnings("unchecked")어노테이션 사용시 항상 경고를 무시해도 안전한 이유를 주석으로 작성해야함

---

비검사 경고는 중요하니 무시하지 말자. 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거해라. 경고를 없앨 방법을 찾기 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨겨라. 그런다음 숨기기로한 근거를 주석으로 남겨라.
