# 애그리거트

## 애그리거트

복잡한 도메인을 이해하고 관리하기 쉬운 단위로 만들려면 상위 수준에서 모델을 조망할 수 있는 방법이 필요한데, 그 방법이 바로 애그리거트다. <br>
애그리거트는 관련된 객체를 하나의 군으로 묶어 준다. 수많은 객체를 애그리거트로 묶어서 바라보면 상위 수준에서 도메인 모델 간의 관계를 파악할 수 있다. <br>
애그리거트는 관련된 모델을 하나로 모았기 때문에 한 애그리거트에 속한 객체는 유사하거나 동일한 라이프 사이클을 갖는다. <br>
도메인 규칙에 따라 최초 주문 시점에 일부 객체를 만들 필요가 없는 경우도 있지만 애그리거트에 속한 구성요소는 대부분 함께 생성하고 함께 제거한다.

애그리거트는 경계를 갖는다. 한 애그리거트에 속한 객체는 다른 애그리거트에 속하지 않는다. <br>
애그리거트는 독립된 객체 군이며 각 애그리거트는 자기 자신을 관리할 뿐 다른 애그리거트를 관리하지 않는다. <br>
경계를 설정할 때 기본이 되는 것은 도메인 규칙과 요구사항이다. 도메인 규칙에 따라 함께 생성되는 구성요소는 한 애그리거트에 속할 가능성이 높다.

<img width="712" alt="스크린샷 2025-01-06 오후 8 10 26" src="https://github.com/user-attachments/assets/51fba299-5c9a-4da8-aa3d-8ce3e7dc6221" />

## 애그리거트 루트

애그리거트는 여러 객체로 구성되기 때문에 한 객체만 상태가 정상이면 안 된다. 도메인 규칙을 지키려면 애그리거트에 속한 모든 객체가 정상 상태를 가져야 한다. <br>
주문 애그리거트에서는 OrderLine을 변경하면 Order의 totaLAmounts도 다시 계산해서 총 금액이 맞아야 한다. <br>
**애그리거트에 속한 모든 객체가 일관된 상태를 유지하려면 애그리거트 전체를 관리할 주체가 필요한데, 이 책임을 지는 것이 바로 애그리거트의 루트 엔티티이다.**

<img width="369" alt="스크린샷 2025-01-06 오후 8 14 18" src="https://github.com/user-attachments/assets/992c7415-ca73-419a-965c-43a531d3601b" />

### 도메인 규칙과 일관성

애그리거트 루트의 핵심 역할은 애그리거트의 일관성이 깨지지 않도록 하는 것이다. <br>
이를 위해 애그리거트 루트는 애그리거트가 제공해야 할 도메인 기능을 구현한다. <br>
예를 들면 주문 애그리거트는 배송지 변경, 상품 변경과 같은 기능을 제공하고, 애그리거트 루트인 Order가 이 기능을 구현한 메서드를 제공한다.

**불필요한 중복을 피하고 애그리거트 루트를 통해서만 도메인 로직을 구현하게 만들려면 도메인 모델에 대해 다음의 두 가지를 습관적으로 적용해야 한다.**

- 단순히 필드를 변경하는 set 메서드는 public으로 만들지 않는다.
- 밸류 타입은 불변으로 구현한다.

### 트랜잭션 범위

한 트랜잭션에서는 한 개의 애그리거트만 수정해야 한다. <br>
한 트랜잭션에서 두 개 이상의 애그리거트를 수정하면 트랜잭션 충돌이 발생할 가능성이 더 높아지기 때문에 한 번에 수정하는 애그리거트 개수가 많아질수록 전체 처리량이 떨어지게 된다. <br>
한 트랜잭션에서 한 애그리거트만 수정한다는 것은 애그리거트에서 다른 애그리거트를 변경하지 않는다는 것을 의미한다. <br>
애그리거트는 최대한 서로 독립적이어야 하는데 한 애그리거트가 다른 애그리거트의 기능에 의존하기 시작하면 애그리거트 간 결합도가 높아진다. <br>
결합도가 높아지면 높아질수록 향후 수정 비용이 증가하므로 애그리거트에서 다른 애그리거트의 상태를 변경하지 말아야 한다.

## 리포지터리와 애그리거트

애그리거트는 개념상 완전한 한 개의 도메인 모델을 표현하므로 객체의 영속성을 처리하는 리포지터리는 애그리거트 단위로 존재한다. <br>
Order와 OrderLine을 물리적으로 각각 별도의 DB 테이블에 저장한다고 해서 Order와 OrderLine을 위한 리포지터리를 각각 만들지 않는다. <br>
Order가 애그리거트 루트고 OrderLine은 애그리거트에 속하는 구성요소이므로 Order를 위한 리포지터리만 존재한다.

```java
// 리포지터리에 애그리거트를 저장하면 애그리거트 전체를 영속화해야 한다.
orderRepository.save(order);

// 리포지터리는 완전한 order를 제공해야 한다.
Order order = orderRepository.findById(orderId);

// order가 온전한 애그리거트가 아니면
// 기능 실행 도중 NullPointerException 같은 문제가 발생한다.
order.cancel();
```

## ID를 이용한 애그리거트 참조

한 객체가 다른 객체를 참조하는 것처럼 애그리거트도 다른 애그리거트를 참조한다. <br>
애그리거트 관리 주체는 애그리거트 루트이므로 애그리거트에서 다른 애그리거트를 참조한다는 것은 다른 애그리거트의 루트를 참조한다는 것과 같다. <br>
예를 들어 주문 애그리거트에 속한 Orderer는 주문한 회원을 참조하기 위해 회원 애그리거트 루트인 Member를 필드로 참조할 수 있다.

애그리거트를 직접 참조할 때 발생할 수 있는 가장 큰 문제는 편리함을 오용할 수 있다는 것이다. <br>
**한 애그리거트 내부에서 다른 애그리거트 객체에 접근할 수 있으면 다른 애그리거트의 상태를 쉽게 변경할 수 있게 된다.**

<img width="678" alt="스크린샷 2025-01-06 오후 8 26 48" src="https://github.com/user-attachments/assets/65e68212-3c2e-407d-b897-c9d39b72d4a2" />

ID 참조를 사용하면 모든 객체가 참조로 연결되지 않고 한 애그리거트에 속한 객체들만 참조로 연결된다. <br>
이는 애그리거트의 경계를 명확히 하고 애그리거트 간 물리적인 연결을 제거하기 때문에 모델의 복잡도를 낮춰준다. 또한 애그리거트 간의 의존을 제거하므로 응집도를 높여주는 효과도있다.












