# DDD와 전술 패턴
로직이 복잡한 도메인을 구현할 때는 DDD(Domain Driven Design)에 소개된 패턴을 사용하는 것을 검토해봐야 한다.

DDD에서 소개하는 전술 패턴을 사용하면 도메인 영역에 도메인 로직을 집중시키는 데 도움이 된다.

## DDD에서의 도메인 모델의 구성 요소
- 엔티티(Entity) : 각 엔티티 객체는 고유의 식별자를 가지며, 각 엔티티는 식별자로 구분된다. 내부 상태가 바뀌어도 식별자는 바뀌지 않는다.
    - 예로 각 주문 엔티티는 서로 다른 주문번호를 식별자로 갖는다.

- 벨류(Value) : 밸류는 고유의 식별자를 갖지 않으며 개념적인 값을 포함한다. 금액, 배송 주소 같은 값이 밸류가된다. 값은 불변으로 구현하는 것을 추천한다.

- 애그리거트(Aggregate) : 애그리거트는 관련된 객체를 묶어 하나의 개념적인 단위를 표현한다. 모델의 일관성을 관리하는 단위가 된다.
    - 예로 주문 애그리거트는 Order엔티티, OrderLine 밸류 집합, ShippingAddress밸류로 구성될 수 있다.

- 리포지토리(Repository) : 도메인 객체를 물리적인 저장소와 연결할 때 사용하는 모델이 리포지토리이다. 도메인 객체를 저장하고 조회할 때 사용하는 인터페이스를 제공한다. 레포지토리는 애그리거트 단위로 존재한다.

- 도메인서비스(Domain Service) : 특정한 애그리거트에 속하지 않은 로직을 구현한다. 외부 연동이 필요한 도메인 로직도 도메인 서비스로 사용해서 표현한다.

- 도메인이벤트(Domain Event) : 도메인 내에서 발생한 이벤트를 표현한다. 도메인의 상태가 변경될 때 도메인 이벤트가 발생한다. 도메인 이벤트는 주로 다른 부분에 변화를 알리기 위해 사용된다.

DDD는 구성 요소를 사용해서 도메인 로직을 애그리거트 단위로 묶는다 복잡한 모델을 애그리거트 단위로 관리할 수 있게 함으로써 복잡도를 낮추고 애그리거트에 관련 로직을 모아 응집도를 높인다.

```java
// DDD의 전술 패턴은 도메인 로직을 한 곳에 모으는 데 도움이 된다.
public class CancelOrderService{
    private OrderRepository orderRepository;

    @Transactional
    public void cancel(OrderNumber orderNum){
        // 응용 서비스는 도메인 모델을 사용해서 사용자 요청을 처리한다.
        Optional<Order> orderOpt = orderRepository.findById(orderNum);
        Order order = orderOpt.orElseThrow(() -> new NoOrderException());
        // 주문 취소 로직은 Order 애그리커드에 위치한다.
        order.cancel();
    }
}
```

전술 패턴 외에도 바운디드 컨텍스트는 도메인 간의 경계를 설정해준다. 이는 상위 수준에서 복잡한 도메인을 관리하는데 도움을 주며, 이어서 살펴볼 MSA와도 잘어울린다.