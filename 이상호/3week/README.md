# [Unit Testing] 3장 단위 테스트 구조

이 내용은 [단위 테스트 생산성과 품질을 위한 단위 테스트 원칙과 패턴]을 읽으면서 정리한 내용을 포함하고 있습니다.

- 3장 단위 테스트 구조  3.1 ~ 3.4

목차는 다음과 같습니다.

- 단위 테스트를 구성하는 방법
- xUnit 테스트 프레임워크 살펴보기
- 단위 테스트 명명법
- 정리

## 단위 테스트 구조

여기에서는 유용하지 않는 단위 테스트 메소드 명명 사례를 소개하고 왜 좋은 선택이 아닌지 살펴본다. 그리고 테스트를 작성한 개발자 외 문제 영역에 익숙한 다른 사람도 쉽게 읽을 수 있는 방식의 테스트 명명법을 간단하고 따라 하기 쉽게 설명한다.
그리고 단위 테스트를 간소화하는 프레임워크의 기능도 포함한다.

### 단위 테스트를 구성하는 방법

여기에서는 다음을 알아본다.

- 준비, 실행 검증 패턴을 사용해 단위 테스트를 구성하는 방법
- 피해야 할 함정
- 테스트를 가능한 한 읽기 쉽게 만드는 방법

#### AAA 패턴

- 각 테스트를 준비, 실행, 검증이라는 세 부분으로 나눈다.

예제를 통해 알아보자.

```java
public class Calculator {

    public double sum(final double first, final double second) {
        return first + second;
    }
}
```

AAA 패턴을 적용한 테스트 코드는 다음과 같다.

```java
class CalculatorTest {

    @Test
    void sum_of_two_numbers() {
        // 준비
        double first = 10;
        double second = 20;
        final var calculator = new Calculator();

        // 실행
        double result = calculator.sum(first, second);

        // 검증
        assertThat(result).isEqualTo(30);
    }
}
```

- `CalculatorTest` : 응집도 있는 테스트 세트를 위한 클래스
- `sum_of_two_numbers` : 단위 테스트 이름
- 각 준비, 실행, 검증 구절

##### AAA 패턴의 장점

- 스위트 내 모든 테스트가 단순하고 균일한 구조를 갖어서 일관성이 가장 큰 장점
- 익숙해지면 모든 테스트를 쉽게 읽을 수 있고 이해 가능
- 전체 테스트 스위트의 유지 보수 비용이 줄어듬

###### AAA 패턴 구조

- 준비 구절
  - 테스트 대상 시스템(`SUT`)과 해당 의존성을 원하는 상태로 만든다.
- 실행 구절
  - `SUT`에서 메서드를 호출하고 준비된 의존성을 전달하며 출력 값을 캡쳐한다.
- 검증 구절
  - 결과를 검증하는데, 결과는 반환 값이나 `SUT`와 협력자의 최종 상태, `SUT`가 협력자에 호출한 메서드 등으로 표시될 수 있다.

> Given-When-Then 패턴
> AAA 패턴과 유사한 구조이다.

###### 테스트 작성 방법

- 테스트 코드 작성할 때
  - 준비 구절부터 시작하는 것이 자연스럽다.
- 테스트 주도 개발(`TDD`) 실천할 때
  - 검증 구절부터 시작하는 것이 자연스럽다.
  - 기능을 개발하기 전에 실패할 테스트를 만들 때는 먼저 기대하는 동작으로 윤곽을 잡은 다음, 이러한 기대에 부응하기 위한 시스템을 어떻게 개발할지 아는 것이 좋다.
  - 특정 동작이 무엇을 해야 하는지에 대한 목푤르 생각하면서 시작한다. 그리고 그 다음이 실제 문제 해결이다.

#### 여러 개의 준비, 실행, 검증 구절 피하기

준비, 실행 또는 검증 구절이 여러 개 있는 테스트를 만날 수 있다.

- 검증 구절이 구분된 여러 갱의 실행 구절을 보면, 여러 개의 동작 단위를 검증하는 테스트를 뜻한다.
- 이러한 테스트는 더 이상 단위 테스트가 아니라 통합 테스트다.
- 이러한 통합 테스트는 피하는 것이 좋다.

실행이 하나인 경우 테스트의 장점은 다음과 같다.

- 테스트가 단위 테스트 범주에 있게끔 보장하고, 간단하고, 빠르며, 이해하기 쉽다.

통합 테스트에서는 실행 구절을 여러 개 두는 것이 괜찮을 때도 있다.

- 통합 테스트는 느릴 수 있기 때문에, 속도를 높이는 한 가지 방법은 여러 개의 통합 테스트를 여러 실행과 검증이 있는 단일한 테스트로 묶는 것이다.

##### Action

- 일련의 실행과 검증이 포함된 테스트를 본다면 리팩토링해야 한다.
- 각 동작을 고유의 테스트로 도출해야 한다.

#### 테스트 내 if 문 피하기

if 문이 있는 단위 테스트는 안티 패턴이다.

- 단위 테스트든 통합 테스트든 테스트는 분기가 없는 간단한 일련의 단계여야 합니다.
- 테스트가 한 번에 너무 많은 것을 검증한다는 표시이다. 
- 이러한 테스트는 반드시 여러 테스트로 나눠야 한다.

if 문이 있는 단위 테스트는 테스트를 읽고 이해하는 것을 더 어렵게 만들게 된다.

##### Action

- if 문 보다는 케이스를 나눠서 작성한다.

#### 각 구절은 얼마나 커야 하는가?

준비 구절에서 재사용에 도움이 되는 두 가지 패턴이 있다.

- 오브젝트 마더
- 테스트 데이터 빌더

실행 구절은 보통 코드 한 줄이다. 

- 실행 구절이 두 줄 이상인 경우 문제가 있을 수 있다.

다음은 한 줄로 실행된 구절입니다.

```java
@Test
void purchase_succeeds_when_enough_inventory() {

    Store store = new Store();
    store.addInventory(Product.Shampoo, 10);        Customer customer = new Customer();

    boolean success = customer.purchase(store, Product.Shampoo, 5);

    assertAll(
        () -> assertThat(success).isTrue(),
        () -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(5)
    );
}
```

다음은 두 줄로 실행된 구절입니다.

```java
@Test
void purchase_succeeds_when_enough_inventory_2() {

    // 준비
	Store store = new Store();
	store.addInventory(Product.Shampoo, 10);
	Customer customer = new Customer();

    // 실행
	boolean success = customer.purchase(store, Product.Shampoo, 5);
	store.removeInventory(success, Product.Shampoo, 5);

    // 검증
	assertAll(
        () -> assertThat(success).isTrue(),
		() -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(5)
	);
}
```

- 구매를 마치려면 두 번째 메서드를 호출해야 하므로, 캡슐화가 깨지게 된다.
- 비즈니스 관점에서 구매가 정상적으로 이뤄지면 고객의 재품 획득과 매장 재고 감소라는 두 가지 결과가 만들어지게 된다.
  - 이러한 결과는 같이 만들어야 하고 이는 단일한 공개 메서드가 있어야 한다.

이러한 모순을 불변 위반이라고 하며, 잠재적 모순으로부터 코드를 보호하는 행위를 캡슐화라고 한다. 

##### Action

- 해결책은 코드 캡슐화를 항상 지키는 것을 신경써야 한다.
- 불변을 지키는 한, 불변 위반을 초래할 수 있는 잠재적인 행동을 제거해야 한다.

#### 검증 구절에는 검증문이 얼마나 있어야 하는가?

검증 구절이 너무 커지는 것은 경계해야 한다.

- 하나의 테스트로 그 모든 결과를 평가하는 것이 좋다.

##### Action

- SUT에서 반환된 객체 내에서 모든 속성을 검증하는 대신 객체 클래스 내에 적절한 동등 멤버를 정의하는 것이 좋다.

#### 종료 단계는 어떤가

준비, 실행, 검증 이후의 네 번째 구절로 종료 구절을 따로 구분하기도 한다.

- 테스트에 의해 작성된 파일을 지우거나 데이터 베이스 연결을 종료하고자 이 구절을 사용할 수 있다.
- 일반적으로 별도의 메서드로 도출돼, 클래스 내 모든 테스트에서 재사용된다.
- AAA 패턴에는 이 단계를 포함하지 않는다.

#### 테스트 대상 시스템 구별하기

SUT는 테스트에서 중요한 역할을 하는데, 애플리케이션에서 호출하고자 하는 동작에 대한 진입점을 제공한다.

- 진입점은 동작을 수행할 하나의 클래스

SUT를 의존성과 구분하는 것이 중요하다.

- SUT가 꽤 많은 경우, 테스트 대상을 찾는 데 시간을 너무 많이 들일 필요가 없다.

##### Action

- 그렇게 하기 위해 테스트 내 SUT 이름을 sut로 하라

```java
class CalculatorTest {

    @Test
    void sum_of_two_numbers() {
        // 준비
        double first = 10;
        double second = 20;
        final var sut = new Calculator();

        // 실행
        double result = sut.sum(first, second);

        // 검증
        assertThat(result).isEqualTo(30);
    }
}
```

#### 준비, 실행, 검증 주석 제거하기

테스트 내에서 특정 부분이 어떤 구절에 속해 있는지 파악하는 데 시간을 많이 들이지 않도록 세 구절을 서로 구분하는 것 역시 중요하다.

- 이를 위한 한가지 방법은 각 구절을 시작하기 전에 주석을 다는 것이다.

##### Action

- 주석보다는 빈 줄로 분리하는 것이 좋다.

```java
class CalculatorTest {

    @Test
    void sum_of_two_numbers() {
        double first = 10;
        double second = 20;
        final var sut = new Calculator();

        double result = sut.sum(first, second);

        assertThat(result).isEqualTo(30);
    }
}
```

### xUnit 테스트 프레임워크 살펴보기

- 생략

### 테스트 간 테스트 픽스쳐 재사용

테스트에서 언제 어떻게 코드를 재사용하는지 아는 것이 중요하다. 준비 구절에서 코드를 재사용하는 것이 테스트를 줄이면서 단순화하기 좋은 방법이다.

- 준비 구절에서는 별도의 메서드나 클래스로 도출한 후 테스트 간에 재사용하는 것이 좋다.

#### 테스트 픽스쳐

- 테스트 픽스쳐는 테스트 실행 대상 객체이며, 이 객체는 정규 의존성, 즉 SUT로 전달되는 인수다.
- 테스트 픽스쳐는 데이터베이스에 있는 데이터나 하드 디스크의 파일일 수도 있다.
- 테스트 실행 전 고정 상태를 유지한다.

아래는 `@BeforeEach` 을 이용해서 테스트 픽스쳐를 초기화하고 있다.

```java
class CustomerTest {

    private Store store;
    private Customer sut;

    @BeforeEach
    void setUp() {

        store = new Store();
        store.addInventory(Product.Shampoo, 10);
        sut = new Customer();
    }

    @Test
    void purchase_succeeds_when_enough_inventory() {

        boolean success = sut.purchase(store, Product.Shampoo, 5);

        assertAll(
            () -> assertThat(success).isTrue(),
            () -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(5)
        );
    }

    @Test
    void purchase_fails_when_not_enough_inventory() {

        boolean success = sut.purchase(store, Product.Shampoo, 15);

        assertAll(
            () -> assertThat(success).isFalse(),
            () -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(10)
        );
    }
}
```

테스트 코드의 양을 크게 줄일 수 있으며, 테스트에서 테스트 픽스쳐 구성을 전부 또는 대부분 제거할 수 있다. 그러나 큰 단점이 있다.

- 테스트 간 결합도가 높아진다.
- 테스트 가독성이 떨어진다.

#### 테스트 간의 높은 결합도는 안티 패턴이다

모든 테스트가 서로 결합돼 있다. 즉, 테스트의 준비 로직을 수정하면 클래스의 모든 테스트에 영향을 미친다.

예를 들어,

```java
store.addInventory(Product.Shampoo, 10);
```

위 코드를 다음과 같이 수정한다면,

```java
store.addInventory(Product.Shampoo, 15);
```

갑자기 모든 테스트가 실패하게 될 수 있다. 테스트를 수정해도 다른 테스트에 영향을 주어서는 안된다.

- 테스트는 서로 격리돼 실행해야 한다는 것과 비슷하다.

이 지침을 따르려면 테스트 클래스에 공유 상태를 두지 말아야 한다.

#### 테스트 가독성을 떨어뜨리는 생성자 사용

준비 코드를 생성자로 추출할 때의 또 다른 단점은 테스트 가독성을 떨어뜨리는 것이다. 

- 테스트만 보고는 더 이상 전체 그림을 볼 수 없다.

준비 로직이 별로 없더라도(픽스쳐의 인스턴스화만 있을 때) 테스트 메서드로 바로 옮기는 것이 좋다.

#### 더 나은 테스트 픽스쳐 재사용법

테스트 픽스쳐를 재사용할 때 생성자 사용이 최선의 방법은 아니다. 두 번째 방법은 다음 예제와 같이 테스트 클래스에 비공개 팩토리 메서드를 두는 것이다.

```java
class CustomerTest {

    @Test
    void purchase_succeeds_when_enough_inventory() {

        Store store = createStoreWithInventory(Product.Shampoo, 10);
        Customer sut = createCustomer();

        boolean success = sut.purchase(store, Product.Shampoo, 5);

        assertAll(
                () -> assertThat(success).isTrue(),
                () -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(5)
        );
    }

    @Test
    void purchase_fails_when_not_enough_inventory() {

        Store store = createStoreWithInventory(Product.Shampoo, 10);
        Customer sut = createCustomer();

        boolean success = sut.purchase(store, Product.Shampoo, 15);

        assertAll(
                () -> assertThat(success).isFalse(),
                () -> assertThat(store.findInventory(Product.Shampoo)).isEqualTo(10)
        );
    }

    private Store createStoreWithInventory(Product product, int quantity) {

        Store store = new Store();
        store.addInventory(product, quantity);
        return store;
    }

    private static Customer createCustomer() {

        return new Customer();
    }
}
```

##### Action

- 테스트에 픽스쳐를 어떻게 생성하지 지정할 수 있다.
- 팩토리 메서드를 통해 상점에 샴푸 열 개를 추가하라고 테스트에 명시돼 있다. 이는 매우 읽기 쉽고 재사용이 가능하다. 생성된 상점의 특성을 이해하려고 팩토리 메서드 내부를 알아볼 필요가 없기 때문에 가독성이 좋다.

### 단위 테스트 명명법

테스트에 표현력이 있는 이름을 붙이는 것이 중요하다.

- 올바른 명칭은 테스트가 검증하는 내용과 기본 시스템의 동작을 이해하는 데 도움이 된다.

가장 유명하지만 가장 도움이 되지 않는 방법 : `[테스트 대상 메서드]_[시나리오]_[예상 결과]`

- 테스트 대상 메서드 : 테스트 중인 메서드의 이름
- 시나리오 : 메서드를 테스트하는 조건
- 예상 결과 : 현재 시나리오에서 테스트 대상 메서드에 기대하는 것

동작 대신 구현 세부 사항에 집중하게끔 부추기기 때문에 분명히 도움이 되지 않는다.

위에서 계산기 객체의 테스트 명명 규칙을 가장 유명한 방법으로 하면 다음과 같다.

- `sum_twoNumbers_returnsSum`
  - `sum` 은 왜 테스트 이름으로 두번이 나타나는가?
  - `returns` 합계는 어디로 반환되는가?

간단하고 쉬운 영어 구문이 훨씬 더 효과적이며, 엄격한 명명 구조에 얽매이지 않고 표현력이 뛰어나다.

- `sum_of_two_numbers`
  - 쉬운 영어로 작성한 위의 이름이 읽기에 훨씬 간단하다.

#### 단위 테스트 명명지침

표현력이 있고 읽기 쉬운 테스트 이름을 짓는 방법은 다음과 같다.

- 엄격한 명명 정책을 따르지 않는다. 표현의 자유를 허용한다.
- 문제 도메인에 익숙한 기밸자들에게 시나리오를 설명하는 것처럼 테스트 이름을 짓는다. 도메인 전문가나 비즈니스 분석가가 좋은 예다.
- 단어를 밑줄 표시로 구분한다. 그러면 특히 긴 이름에서 가독성을 향상시키는 데 도움이 된다.

테스트 클래스의 이름을 지을 때 밑줄 표시를 사용하지 않는다.

- 보통 클래스 이름은 그리 길지 않아서 밑줄 표시가 없어도 잘 읽을 수 있다.

#### 예제 : 지침에 따른 테스트 이름 변경

테스트 하나를 예로 들어서 방금 설명한 지침으로 천천히 이름을 개선해보자.

- 다음 예제는 과거 배송일이 유효하지 않다는 것을 검증하는 테스트이다.

다음의 테스트는 `DeliveryService` 가 잘못된 날짜의 배송을 올바르게 식별하는지 검증한다.

```java
class DeliveryServiceTest {

    @Test
    void isDeliveryValid_invalidDate_returnsFalse() {

        DeliveryService sut = new DeliveryService();
        LocalDate passDate = LocalDate.now().plusDays(1);
        Delivery delivery = new Delivery(passDate);

        boolean isValid = sut.isDeliveryValid(delivery);

        assertThat(isValid).isFalse();
    }
}
```

해당 테스트 이름을 쉬운 영어로 어떻게 다시 작성할까?

- `delivery_with_invalid_date_should_be_considered_invalid()`

새로운 버전에서 두 가지가 눈에 띈다.

- 이제 이름이 프로그래머가 아닌 사람들에게 납득되고, 프로그래머도 더 쉽게 이해할 수 있다.
- SUT의 메서드 이름은 더 이상 테스트명에 포함되지 않는다.

여전히 이상적이지 않지만, 너무 장황하다. `considered` 라는 단어를 제고해도 
의미가 퇴색되지 않는다.

- `delivery_with_past_date_should_be_invalid`

`should be` 문구는 또 다른 일반적인 안티 패턴이다. 테스트는 동작 단위에 대해 단순하고 원자적인 사살이라고 했다. 사실을 서술할 때는 소망이나 욕구가 들어가지 않는다.

- `delivery_with_past_date_is_invalid`

기초 영문법을 지켜야 한다. 관사를 붙이면 테스트를 완벽하게 읽을 수 있다.

- `delivery_with_a_past_date_is_invalid`

## 정리

- 단위 테스트를 구성하는 방법에는 다음과 같은 내용이 있다.
  - AAA 패턴
    - 모든 테스트가 단순하고 균일한 구조를 갖어서 일관성이 가장 큰 장점이다.
  - 여러 개의 준비, 실행, 검증 구절 피하기
    - 테스트가 단위 테스트 범주에 있게끔 보장하고, 간단하고, 빠르며, 이해하기 쉽다.
  - 테스트 내 if 문 피하기
    - 테스트가 한 번에 너무 많은 것을 검증한다는 표시이기 때문에 이러한 테스트는 반드시 여러 테스트로 나눠야 한다.
  - 각 구절은 얼마나 커야 하는가?
    - 잠재적 모순으로부터 코드를 보호해야 하며, 단일한 공개 메서드가 필요하다.
  - 검증 구절에는 검증문이 얼마나 있어야 하는가?
    - 하나의 테스트로 그 모든 결과를 평가하는 것이 좋은데, 객체간 비교를 통해 해결한다.
  - 종료 단계는 어떤가
    - 일반적으로 별도의 메서드로 도출돼, 클래스 내 모든 테스트에서 재사용할때 사용한다.
  - 테스트 대상 시스템 구별하기
    - SUT는 테스트에서 중요한 역할을 하는데, 애플리케이션에서 호출하고자 하는 동작에 대한 진입점을 제공한다.
  - 준비, 실행, 검증 주석 제거하기
    - 각 구절을 시작하기 전에 주석을 다는 것보다 빈 칸으로 구분한다.
- 테스트 간 테스트 픽스쳐 재사용
  - 팩토리 메서드를 통해 팩토리 메서드 내부를 알아볼 필요가 없기 때문에 가독성이 좋다.
- 단위 테스트 명명법
  - 테스트에 표현력이 있는 이름을 붙이는 것이 중요하다.  