# JPA 실전 스프링부트

---

- 도메인 분석 설계
- 애풀리케이션 구현 준비
- 회원 도메인 개발
- 상품 도메인 개발
- 주문 도메인 개발
- 웹 계층 개발

## 도메인 분석 설계

**요구사항 분석**

실제 동작하는 화면을 먼저 확인한다.

기능목록 작성

![image](https://user-images.githubusercontent.com/109144975/216894869-baf11793-3de1-443b-b78a-7b8cbc9b13a7.png)
- 회원 기능
  - 회원등록
  - 회원조회
- 상품 기능
  - 상품 등록
  - 상품 수정
  - 상품 조회
- 주문 기능
  - 상품 주문
  - 주문 내역 조회
  - 주문 취소
- 기타 요구사항
  - 상품은 재고 관리가 필요하다.
  - 상품의 종류는 도서, 음반, 영화가 있다.
  - 상품을 카테고리로 구분할 수 있다.
  - 상품 주문시 배송 정보를 입력할 수 있다.

**도메인 모델과 테이블 설계**

![](https://user-images.githubusercontent.com/109144975/216894890-0712b787-6e68-43f0-b7b8-a9b89b9d1d2b.png)

회원, 주문, 상품의 관계 :

- 회원은 여러 상품 주문이 가능하다.
- 한 번 주문 시 여러 상품 선택이 가능하므로 주문과 상품은 다대다 관계이다.
- 다대다 관계는 위 그림처럼 주문상품의 엔티티를 추가하여 다대일 관계로 풀어내야한다.

상품 분류 : 상품은 도서, 음반, 영화로 구분되는데 상품이라는 공통 속성을 사용하므로 상속구조로 표현했다.


**회원 엔티티 분석**


![](https://user-images.githubusercontent.com/109144975/216894940-cf259806-305b-4073-b969-c7a1072f54c7.png)

회원(Member) : 이름과 임베디드 타입인 주소, 그리고 주문(orders)리스트를 가진다.

주문(Order) : 한 번 주문시 여러 상품을 주문할 수 있으므로 주문과 주문상품(OrderItem)은 일대다 관계다. 주문은 상품을 주문한 회원과 배송 정보
, 주문 날짜, 주문 상태(status)를 가지고 있다. 주문 상태는 열거형을 사용했는데 주문(ORDER), 취소(CANCEL)을 표현할 수 있다.

주문상품(OrderItem) : 주문한 상품 정보와 주문 금액(orderPrice), 주문 수량(count)정보를 가지고 있다.

상품(Item) : 이름, 가격, 재고수량(stockQuantity)을 가지고 있다. 상품을 주문하면 재고수량이 줄어든다. 상품의 종류는 도서, 음반, 영화가 있는데 
각각은 사용하는 속성이 다르다.

배송(Delivery) : 주문시 하나의 배송 정보를 생성한다. 주문과 배송은 일대일 관계다.

카테고리(Category) : 상품과 다대다 관계를 맺는다. parent, child 로 부모, 자식 카테고리를 연결한다.

주소(Address) : 값 타입이다. 회원과 배송에서 사용한다.

> 참고 : 회원이 주문을 하기 때문에, 회원이 주문리스트를 가지는 것은 얼핏 보면 잘 설계한 것 같지만, 객체 세상은 실제 세계와는 다르다. 실무에서는 
회원이 주문을 참조하지 않고, 주문이 회원을 참조하는 것으로 충분하다. 여기서는 일대다, 다대일의 양방향 연관관계를 설명하기 위해 추가했다. 


**회원 테이블 분석**

![](https://user-images.githubusercontent.com/109144975/216894972-31b29cd8-1f8a-474a-9630-905a577fef2f.png)

모든 엔티티에서 해당 엔티티의 ID는 공통으로 가지고 있다. (가장위에 표시) 
연관관계를 FK(foreign key) 중심적으로 생각해보자

MEMBER : 회원 엔티티의 Address 임베디드 타입 정보가 회원 테이블에 그대로 들어갔다. 이건 DELIVERY 테이블도 마찬가지다.

ITEM : 앨범, 도서, 영화 타입을 통해서 하나의 테이블로 만들었다. DTYPE 컬럼으로 타입을 구분한다.

ORDERS : ORDER은 MEMBER와의 관계에서 일대다 중 다 이다. MEMBER_ID(FK) 를 가지게 하여 ORDERS를 연관관계의 주인으로 만들어 주었다.
연관관계의 주인이란 두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 이것을 연관관계의 주인이라고 한다. * DELIVERY도 마찬가지 이다.

> - 연관관계의 주인만이 데이타베이스 연관관계와 매핑된다.
> - 연관관계의 주인만이 외래키를 관리(등록, 수정, 삭제)할 수 있다.
> 주인이 아닌 쪽은 읽기만 할 수 있다.
> - 연관관계의 주인을 정한다는 것 = 외래 키 관리자를 선택하는 것.

**연관관계 매핑 분석**

회원과 주문 : 일대다, 다대일 양방향 관계다. 따라서 외래 키가 있는 주문(Order)을 연관관계의 주인으로 정하는 것이 좋다. 그러므로 Order.member를 ORDERS.MEMBER_ID
외래 키와 매핑한다.

주문 상품과 주문 : 다대일 양방향 관계다. 외래 키가 주문상품에 있으므로 주문상품이 연관관계의 주인이다. 그러므로 OrderItem.order를 ORDER_ITEM.ORDER_ID
외래 키와 매핑한다.

주문상품과 상품 : 다대일 단방향 관계다. OrderItem.item을 ORDER_ITEM.ITEM_ID 외래 키와 매핑한다.

주문과 배송 : 일대일 양방향 관계다. Order.delivery를 ORDERS.DELIVERY_ID 외래 키와 매핑한다.


**엔티티 클래스 개발**

- 예제에서는 설명을 쉽게 하기 위해 엔티티 클래스에 Getter, Setter을 모두 열고, 단순하게 설계
- 실무에서는 가급적 Getter만 열어두고, Setter는 꼭 필요한 경우에만 사용

엔티티 코드는 README 가 아닌 코드를 참고하자

> 참고 : Getter는 아무리 호출해도 어떤 일이 발생하지 않는다. 하지만 Setter를 호출하면 데이터가 변한다. 그래서 Setter를 막 열어두면 가까운 
> 미래에 엔티티에 왜 변경되었는지 추적하기가 점점 힘들어 진다. 그래서 엔티티를 변경할때는 Setter 대신에 변경지점이 명확하도록 비즈니스 메서드를
> 별도로 제공해야한다.


> 참고: 엔티티의 식별자는 id 를 사용하고 PK 컬럼명은 member_id 를 사용했다. 엔티티는 타입(여기서는
Member )이 있으므로 id 필드만으로 쉽게 구분할 수 있다. 테이블은 타입이 없으므로 구분이 어렵다.
그리고 테이블은 관례상 테이블명 + id 를 많이 사용한다. 참고로 객체에서 id 대신에 memberId 를
사용해도 된다. 중요한 것은 일관성이다.

**엔티티 설계시 주의점**

엔티티에는 가급적 Setter를 사용하지 말자.

모든 연관관계는 지연로딩으로 설정!
- 즉시로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다. 특히 JPQL을 실행시 N+1 문제가 자주 발생한다.
- 실무에서 모든 연관관계는 지연로딩(LAZY)으로 설정해야 한다.
- @XToOne 관계는 기본이 즉시 로딩이므로 직접 지연로딩으로 설정해야 한다.

- ex)
```java
    @OneToOne(mappedBy = "delivery", fetch = LAZY)
    private Order order;
```

컬렉션은 필드에서 초기화 하자

- 컬렉션은 필드에서 초기화 하는 것이 안전하다.
- null 문제에서 안전하다.
- 하이버네이트에서 엔티티를 영속화 할 때, 컬렉션을 감싸서 하이버네이트에 맞게 변경한다. 따라서 필드에서 초기화 하지 않고 생성자에서 초기화를
한다면 문제가 발생할 수 있다.

ex)

```java
Member member = new Member();
System.out.println(member.getOrders().getClass());
em.persist(member);
System.out.println(member.getOrders().getClass());
//출력 결과
class java.util.ArrayList
class org.hibernate.collection.internal.PersistentBag
```
---

## 애플리케이션 구현 준비

**구현 요구사항**

- 회원 기능
  - 회원 등록
  - 회원 조회
- 상품 기능
  - 상품 등록
  - 상품 수정
  - 상품 조회
- 주문 기능
  - 상품 주문
  - 주문 내역 조회
  - 주문 취소


**애플리케이션 아키텍처**

![](https://user-images.githubusercontent.com/109144975/216905479-4e34bd4b-1b11-43ec-81ab-f5e7c9a58dd4.png)

계층형 구조 사용

- controller, web: 웹 계층
- service: 비즈니스 로직, 트랜잭션 처리
- repository: JPA를 직접 사용하는 계층, 엔티티 매니저 사용
- domain: 엔티티가 모여 있는 계층, 모든 계층에서 사용


패키지 구조
- jpabook.jpashop
  - domain
  - exception
  - repository
  - service
  - web

개발순서 : 서비스, 리포지토리 계층을 개발하고, 테스트 케이스를 작성해서 검증, 마지막에 웹 계층 적용

---

## 회원 도메인 개발

구현기능

- 회원등록
- 회원 목록 조회

순서

- 회원 리포지토리 개발
- 회원 서비스 개발
- 회원 기능 테스트

**회원 리포지토리 개발**

기능

- save()
- findOne()
- findAll()
- findByName()

**회원 서비스 개발**

- join()
- findMembers()
- findOne()

## 상품 도메인 개발

**구현 기능**

- 상품 등록
- 상품 목록 조회
- 상품 수정

**순서**

- 상품 엔티티 개발(비즈니스 로직 추가)
- 상품 리포지토리 개발
- 상품 서비스 개발
- 상품 기능 테스트

## 주문 도메인 개발

**구현 기능**

- 상품 주문
- 주문 내역 조회
- 주문 취소

**순서**

- 주문 엔티티, 주문상품 엔티티 개발
- 주문 리포지토리 개발
- 주문 서비스 개발
- 주문 검색 기능 개발
- 주문 기능 테스트

<br>

```java

//==생성 메서드==//
public static OrderItem createOrderItem(Item item, int orderPrice, int count) {
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPrice(orderPrice);
        orderItem.setCount(count);

        item.removeStock(count);
        return orderItem;
        }

//==비즈니스 로직==//
public void cancel() {
        getItem().addStock(count); // 재고 수량 원상 복구
        }
        
```

<br>

```java
    //==생성 메서드==//
public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem: orderItems) {
        order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
        }

//==비즈니스 로직==//
/**
 * 주문 취소
 */
public void cancel() {
        if (delivery.getStatus() == DeliveryStatus.COMP) {
        throw new IllegalStateException("이미 배송완료된 상푸믄 취소가 불가능합니다.");
        }

        this.setStatus(OrderStatus.CANCEL); // order 상태를 cancel 로 변경
        for (OrderItem orderItem : orderItems) {
        orderItem.cancel();
        }
        }
```

<br>

```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;

    /**
     * 주문
     */
    @Transactional
    public Long order(Long memberId, Long itemId, int count) {
        // 엔티티 조회
        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        // 배송정보 생성
        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());

        // 주문 상품 생성
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        // 주문 생성
        Order order = Order.createOrder(member, delivery, orderItem);

        // 주문 저장
        orderRepository.save(order);
        return order.getId();
    }

    /**
     * 취소
     */
    @Transactional
    public void cancelOrder(Long orderId) {
        // 주문 엔티티 조회
        Order order = orderRepository.findOne(orderId);

        // 주문 취소
        /**
         * 여기서 JPA의 큰 장점이 발휘됨
         * 변경된 데이터가 있을때 SQL을 직접 다룬다면 이를 바깥에서 데이터를 끄집어 내서 바꿔줘야함
         * JPA에서는 엔티티의 데이터만 바꾸면 JPA가 알아서 바뀐 변경 포인트들에 대해 변경내역감지(더티체킹)가 일어나면서 데이터베이스에 업데이트 쿼리가 날라감
         *
         */
        order.cancel();
    }

    //검색
//    public List<Order> findOrders(OrderSearch orderSearch) {
//        return orderRepository.findAll(orderSearch);
//    }
}

```
<br>

위 형태의 코드 방식, 예를 들어 주문에서 OrderItem 의 createOrderItem 메소드, Order 의 createOrder 메소드 처럼 서비스의 order 주문을
생성할때 메소드만을 호출하는 방식, 즉 서비스가 아닌 엔티티에 핵심 비즈니스 로직을 몰아넣는 방식을 **도메인 모델 패턴** 이라 한다.
서비스 계층은 단순히 엔티티에 필요한 요청을 위임(메소드 호출)하는 역할을 한다.

이처럼 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것을 도메인 모델 패턴이라고 한다.