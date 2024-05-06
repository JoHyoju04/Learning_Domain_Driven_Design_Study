# 06_복잡한 비즈니스 로직 다루기

# 도메인 모델 패턴

도메인 모델 패턴은 복잡한 비즈니스 로직을 다루기 위한 것이다. CRUD 인터페이스 대신 복잡한 상태 전환, 항상 보호해야 하는 규칙인 비즈니스 규칙과 불변성을 다룬다.

## 구성 요소

### Aggregate

**트랜잭션 경계를 공유하는 엔티티의 계층**이다. 애그리게이트의 경계에 속하는 모든 데이터는 비즈니스 로직의 구현을 통해 강력한 일관성을 유지해야 한다.
**애그리게이트의 상태와 내부 객체는** 애그리게이트의 커맨드를 실행하여 **퍼블릭 인터페이스를 통해서만 수정 될 수 있다.** 애그리게이트와 관련된 모든 비즈니스 로직이 경계 내에 존재하도록 외부 컴포넌트는 애그리게이 트 내의 데이터 필드를 읽을 수만 있게 한다.
애그리게이트는 **트랜잭션의 경계 역할**을 한다. 내부 객체를 포함한 모든 데이터는 원자적인 단일 트랜잭션으로 데이터베이스에 커밋돼야 한다.
애그리게이트는 도메인 이벤트를 게시하여 외부 엔티티와 커뮤니케이션할 수 있다. 도메인 이벤트는 애그리게 이트의 수명주기에서 중요한 비즈니스 이벤트를 설명하는 메시지다. 다른 컴포넌트는 이벤트를 구독하고 비즈 니스 로직의 실행을 촉발하는 데 사용할 수 있다.

```java
public class OrderItem {
    private String productId;
    private int quantity;

    public OrderItem(String productId, int quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }

    // Getters and setters
}

public class Order {
    private String orderId;
    private List<OrderItem> orderItems;
    private OrderStatus status;

    public Order(String orderId) {
        this.orderId = orderId;
        this.orderItems = new ArrayList<>();
        this.status = OrderStatus.CREATED;
    }

    // Add item to order
    public void addItem(String productId, int quantity) {
        this.orderItems.add(new OrderItem(productId, quantity));
    }

    // Update order status
    public void updateStatus(OrderStatus status) {
        this.status = status;
    }

    // Getters and setters
}
```

여기서 **`Order`** 클래스는 주문 애그리게이트를 나타내며, **`OrderItem`** 애그리게이트를 참조하고 있다.  **`OrderStatus`** 는 Value Object라고 할 수 있다.

### Value Object

- 불변 객체로 구현
- 원래 인스턴스를 수정하지 않고 새로운 인스턴스를 생성해서 반환

```java
public class Color{

	public readonly byte Red; 
	public readonly byte Green;
	public readonly byte Blue;
	
	public Color (byte r, byte g, byte b){
	this.Red = r;
	this.Green = g;
	this.Blue = b;
	}
	
	public Color MixWith(Color other){
		return new Color(
			r: (byte) Math. Min(this. Red + other.Red, 255),
			g: (byte) Math. Min (this.Green + other. Green, 255),
			b: (byte) Math. Min (this. Blue + other. Blue, 255)
			);
		}
	}
}
```

참고) Entity

- 식별필드(각 엔티티의 인스턴스마다 고유, 엔티티의 생애주기 내내 불변)
- 밸류 오브젝트는 엔티티의 속성을 설명
- Person이라는 Entity가 있을 때, Address 등등이 value object

### 도메인 서비스

도메인 서비스란 도메인 모델에서 애그리게이트 또는 밸류 오브젝트에 속하지 않는 비즈니스 로직을 담은 상태가 없는 객체다.