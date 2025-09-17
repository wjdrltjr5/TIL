## 타입 안전 이종 컨테이너를 고려하라

---

자바의 단일 원소 컨테이너(Single Element Container)는 기본 데이터 유형(primitive data type)을 객체로 래핑(wrapper)하여 컬렉션과 같은 자료 구조에 저장하기 위한 특별한 형태의 컨테이너입니다.

제네릭은 Set\<E>, Map<K,V> 등의 컬렉션과 ThreadLocal\<T>, AtomicReference\<T>등의 단일 원소 컨테이너에도 흔히 쓰인다. <br>
이런 모든 쓰임에서 매개변수화되는 대상은 (원소가아닌) 컨테이너 자신이다. 따라서 하나의 컨테이너에서 매개변수화할 수 있는 타입의 수가 제한된다. ex) Set에는 원소의 타입을 뜻하는 단 하나의 타입 매개변수만 있으면 되며, Map에는 키와 값을 뜻하는 2개만 필요한 식.

하지만 더 유연한 수단이 필요할 때도 있는법 <br>ex) 데이터베이스의 행(row)은 임의 개수의 열(column)을 가질 수 있는데, 모두 열을 타입 안전하게 이용하는법. <br>
다행이 쉬운 해법이 있다. 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. 이렇게 하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해줄 것이고 이러한 설계 방식을 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)이라 한다.

간단한 예 Favorites 클래스

```java
public class Favorites{//api
  public <T> void putFavorite(Class<T> type, T instance);
  public <T> T getFavorite(Class<T> type);
}

public static void main(String[] args){
  Favorite f = new Favorites();

  f.putFavorite(String.class, "Java");
  f.putFavorite(Integer.class, 0xcafebabe);
  f.putFavorite(Class.class, Favorites.class);

  String favoriteString = f.getFavorite(String.class);
  int favoriteInteger = f.getFavorite(Intefer.class);
  Class<?> favoriteClass = f.getFavorite(Class.class);

  System.out.printf("%s %x %s%n",favoriteString, favoriteInteger, favoriteClass.getName());
}
```

Favorites 인스턴스는 타입 안전하다 String을 요청했는데 Integer를 반환하는 일은 절대 없다.
모든 키의 타입의 각각 제각각이라 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다. 따라서 Favorites는 타입 안전 이종 컨테이너라 할만하다.

```java
public class Favorites{//구현부
  //여기서는 키와 밸류가 타입불일치 할 수 있음
  private Map<Class<?>, Object> favorites = new HashMap<>();

  public <T> void putFavorites(Class<T> type, T instance){
    favorites.put(Objects.requireNonNull(type), instance);
  }
  //여기서 일치여부 확인
  public <T> T getFavorite(Class<T> type){
    return type.cast(favorites.get(type));
  }
}
```

cast 메서드의 반환 타입은 Class객체의 타입 매개변수와 같음

```java
public class Class<T>{
  T cast(Object obj);
}
```

지금의 Favorite 클래스에는 알아두어야 할 제약이 두가지 있다.

-   악의적인 클라이언트가 Class객체를 로타입(item26)으로 넘기면 Favorites 인스턴스의 타입 안전성이 쉽게 깨진다. 하지만 이렇게 짜여진 클라이언트 코드에서는 컴파일할 때 비검사 경고가 뜰것이다.
    Favorites가 타입 불변식을 어기는 일이 없도록 보정하려면 putFavorite메서드에서 동적 형변환을 사용

```java
 public <T> void putFavorites(Class<T> type, T instance){
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
  }
```

-   실체화 불가 타입(item28) 에는 사용할 수 없다. String이나 String[]은 저장가능하지만 List\<String>은 저장할 수 없다. List\<String>용 Class객체를 얻을 수 없기 때문 (로타입 클래스랑 같은 Class객체를 공유)

슈퍼 타입 토큰으로 해결하려는 시도가 있으며 스프링에서는 ParameterizedTypeReference라는 클래스로 미리 구현해 놓음.

```java
Favirotes f = new Favorites();

List<String> pets = Arrays.asList("개", "고양이", "앵무");

f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new TypeRef<List<String>>(){});
```

Favorites가 사용하는 타입 토큰은 비한정적이다. 어떤 Class객체든 받아들인다. 타입을 제한하고 싶다면 한정적 타입 토큰을 활용하면 가능하다.

한정적 타입 토큰이란 한정적 타입 매개변수(item29)나 한정적 와일드 카드(item31)를 사용하여 표현 가능한 타입을 제한하는 토큰

애너테이션 API는 한정적 타입 토큰을 적극적으로 사용한다.
AnnotatedElement메서드는 애너테이션을 런타임에 읽는 기능을 하는 메서드

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

여기서 annotationType인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다. 이 메서드는 토큰으로 명시한 타입의 에너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고 없다면 null을 반환한다. 즉, 애너테이션된 요소는 그 키가 애너테이션 타입인, 타입 안전 이종 컨테이너이다.

Class<?> 타입의 객체가 있고, 이를 (getAnnotation처럼) 한정적 타입 토큰을 받는 메서드에 넘기려면 객체를 Class<? extends Annotation>으로 형변환할 수도 있지만 이 형변환은 비검사이므로 컴파일하면 경고가 발생한다(item27).

Class 클래스가 이런 형변환을 안전하게 동적으로 수행해주는 인스턴스 메서드를 제공한다. 바로 asSubClass 메서드로, 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환한다. (형변환된다는 것은 이 클래스가 인수로 명시한 클래스의 하위 클래스라는 뜻이다.)

```java
static Annotation getAnnotation(AnnotatedElement element,
String annotationTypeName) {
	Class<?> annotationType = null; // 비한정적 타입 토큰
	try {
		annotationType = Class.forName(annotationTypeName);
	} catch (Exception ex) {
		throw new IllegalArgumentException(ex);
	}
	return element.getAnnotation(
		annotationType.asSubclass(Annotation.class));
}
```

---

컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입매개변수의 수가 고정되어 있다. 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다. 타입 안전 이종 컨테이너는 Class를 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한, 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)을 표현한 DatabaseRow타입에는 제네릭 타입인 Column<T>를 키로 사용할 수 있다.
