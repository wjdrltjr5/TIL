# 값 객체를 이용해서 Game 개선하기
- [소스코드](https://github.com/wjdrltjr5/object-principle/tree/main/object-principle-04-03/src/main/java/org/eternity/adventure)

## Game 클래스
```java
public class Game {
    private int width, height; // -> 여러곳에서 사용되며 크기 라는 의미를 내포하고 있음 값객체로 추출
    private Room[] rooms;
    private int x, y;  // ->여러곳에서 사용되며 위치라는 의미를 내포하고 있음 값객체로 추출 
    private boolean running;

    public Game() {
        this.x = 0;
        this.y = 2;
        this.width = 2; 
        this.height = 3;
        this.rooms = arrangeRooms(
                new Room(0, 0, "샘", "아름다운 샘물이 흐르는 곳입니다. 이곳에서 휴식을 취할 수 있습니다."),
                new Room(0, 1, "다리", "큰 강 위에 돌로 만든 커다란 다리가 있습니다."),
                new Room(1, 1, "성", "용왕이 살고 있는 성에 도착했습니다."),
                new Room(0, 2, "언덕", "저 멀리 성이 보이고 언덕 아래로 좁은 길이 나 있습니다."),
                new Room(1, 2, "동굴", "어둠에 잠긴 동굴 안에 작은 화톳불이 피어 있습니다."));
    }
}
```

### 변수뿐 아니라 사용되는 로직도 이동
```java
public class Position {
    private final int x;
    private final int y;
    // 값객체의 경우 정적 팩터리 메서드를 사용하면 가독성이 좋아진다.
    public static Position of(int x, int y) {
        return new Position(x, y);
    }

    private Position(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() {
        return x;
    }

    public int y() {
        return y;
    }

    public Position shift(Direction direction) {
        return switch (direction) {
            case NORTH -> Position.of(x, y - 1); // 숨겨진 개념들을 값객체 내부에서 구현
            case EAST -> Position.of(x + 1, y);
            case SOUTH -> Position.of(x, y + 1);
            case WEST -> Position.of(x - 1, y);
        };
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Position position = (Position) o;
        return x == position.x && y == position.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }

    @Override
    public String toString() {
        return "Position{" +
                "x=" + x +
                ", y=" + y +
                '}';
    }
}

public class Size {
    private final int width, height;

    public static Size with(int width, int height) {
        return new Size(width, height);
    }

    private Size(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public int area() {
        return width * height;
    }

    public boolean contains(Position position) {
        return position.x() >= 0 && position.x() < width &&
                position.y() >= 0 && position.y() < height;
    }

    public int indexOf(Position position) {
        return position.x() + position.y() * width;
    }
}
```

추출된 값객체는 다른 클래스에서 재사용 ex. Room 

일단 값 객체를 만들어 놓으면 복잡한 로직을 담을 수 있는 적합한 장소를 제공받을 수 있다.

![alt text](<구조도.png>)

이렇게 작은것으로 분할한 것들이 쌓이고 쌓여서 훨씬 가독성 높고 유지보수하기 쉬운 코드들을 만들어 낸다.
