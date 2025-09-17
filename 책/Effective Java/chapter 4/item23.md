## 태그 달린 클래스보다는 클래스 계층구조를 활용하라

---

```java
// 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다!
class Figure {

	enum Shape { RECTANGLE, CIRCLE };

	// 태그 필드 - 현재의 모양을 나타낸다.
	final Shape shape;

	// 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
	double length;
	double width;
	// 다음 필드는 모양이 원(CIRCLE)일때만 쓰인다.
	double radius;

	// 원용 생성자
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	// 사각형용 생성자
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	double area() {
		switch(shape) {
			case RECTANGLE:
			return length * width;

			case CIRCLE:
			return Math.PI * (radius * radius);

			default:
			throw new AssertionError(shape);
		}
	}
}
```

태그 달린 클래스에는 단점이 한가득이다.

-   열거 타입 선언, 태그 필드, switch문 등 쓸데 없는 코드가 많다.
-   여러 구현이 한 클래스에 혼합돼 있어서 가독성도 나쁘다.
-   다른 의미를 위한 코드도 언제나 함께 하니 메모리도 많이 사용한다.
-   필드들을final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화 해야 한다.(쓰지않는 필드를 초기화하는 불필요한 코드가 늘어난다.)
-   생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화하는 데 컴파일러가 도와줄 수 있는 건 별로 없다.
-   또 다른 의미를 추가하려면 코드를 수정해야 한다.(예를 들어 새로운 의미를 추가할 때마다 모든 switch문으 찾아 새 의미를 처리하는 코드를 추가해야 하는데 하나라도 빠뜨리면 런타임에 문제가 발생)
-   인스턴스의 타입만으로는 현재 나타내는 의미를 알 길이 전혀 없다.

결론 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.

---

자바와 같은 객체지향언어는 타입 하나로 다양한 의미의 객체를 표현하는 훨씬 나은 수단을 제공한다. 바로 클래스 계측구조를 활용하는 서브타이핑이다.

```java
// 태그 달린 클래스를 클래스 계층 구조로 변환
abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;

	Circle(double radius) { this.radius = radius; }

	@Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
	final double length;
	final double width;

	Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	@Override double area() { return length * width; }
}
```

태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층 구조로 대처하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층 구조로 리팩터링하는 걸 고민해보자
