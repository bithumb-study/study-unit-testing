# Chapter3. 단위 테스트 구조

# 3.1 단위 테스트를 구성하는 방법

## 3.1.1 AAA 패턴 사용

- AAA 패턴(또는 3A 패턴)은 준비(Arrange), 실행(Act), 검증(Assert) 세 부분으로 나눌 수 있다.
- 예제

```java
public class Calculator{
	public double sum(double first, double second)
		return first + second;
}

public class CalculatorTests{
	public void sumOfTwoNumbers(){
		// 준비
		double first = 10;
		double second = 20;
		var calculator = new Calculator();
	
		// 실행
		double result = calculator.sum(first, second);

		// 검증
		Assert.Equal(30, result);
	}

}
```

- AAA 패턴은 모든 테스트가 단순하고 균일한 구조를 갖는 데 도움을 주기에 테스트 스위트의 유지 보수 비용이 줄어든다.
- AAA 패턴의 구조
    - 준비 구절에서는 테스트 대상 시스템과 해당 의존성을 원하는 상태로 만든다
    - 실행 구절에서는 SUT에서 메서드를 호출하고 준비된 의존성을 전달하며 출력 값을 캡처한다.
    - 검증 구절에서는 결과를 검증한다. 결과는 반환 값이나 SUT와 협력자의 최종 상태. SUT가 협력자에 호출한 메서드 등으로 표시될 수 있다.
- 준비 구절부터 시작하는 것이 권장되나 특정 케이스(TDD 실천)에서는 기능이 어떻게 동작할지 충분히 알기 어려울 수 있기에 검증 구절로 시작하는 것이 가능하다.

## 3.1.2 여러 개의 준비, 실행, 검증 구절 피하기

- “테스트 준비 → 실행 → 검증 → 좀 더 실행 → 다시 검증” 과 같이 여러 개의 동작 단위를 검증하는 테스트를 뜻한다
- 이러한 구조는 단위 테스트가 아닌 통합 테스트이기에 되도록 피하는 것이 좋다(리팩토링 필요)
- 실행이 하나면 테스트가 단위 테스트 범주에 있게끔 보장하고, 간단하고, 빠르며, 이해하기 쉽다.

## 3.1.3 테스트 내 if 문 피하기

- if 문은 테스트 한 번에 너무 많은 것을 검증한다는 표시다. 이러한 테스트는 반드시 여러 테스트로 나눠야 한다.

## 3.1.4 각 구절은 얼마나 커야 하는가?

### 준비 구절이 가장 큰 경우

- 일반적으로 준비 구절은 실행과 검증을 합친 만큼 클 수도 있다.
- 하지만 이보다 훨씬 크면 같은 테스트 클래스 내 비공개 메서드 또는 별도의 팩토리 클래스로 도출하는 것이 좋다.

### 실행 구절이 한 줄 이상인 경우를 경계하라

```java
public void purchaseSucceedsWhenEnoughIventory(){
	// 준비
	var store = new Store();
	store.addInventory(Product.Shampoo, 10);
	var customer = new Customer();

	// 실행
	bool success = customer.purchase(store, Product.Shampoo, 5);
	store.RemoveInventory(success, Product.Shampoo, 5);

	// 검증
	Assert.True(success);
	Assert.Equal(5, store.GetInventory(Product.Shampoo));
}
```

위의 예제 실행 수절로 알 수 있는 내용은 다음과 같다

1. 첫 번째 줄에서는 고객이 상점에서 샴푸 다섯 개를 얻으려 한다
2. 두 번째 줄에서 재고가 감소되는데, Purchase() 호출이 성공을 반환하는 경우에만 수행된다.
- 비즈니스 관점에서 구매가 정상적으로 이뤄지면 제품 획득과 매장 재고 감소라는 두 가지 결과가 같이 만들어져야 하고 이는 단일한 공개 메서드가 있어야 한다는 뜻이다.
- 이러한 모순을 **불변 위반**이라하며, 잠재적 모순으로부터 코드를 보호하는 행위를 **캡슐화**라고 한다.

### 3.1.5 검증 구절에는 검증문이 얼마나 있어야 하는가

- 테스트당 하나의 검증을 가질 필요는 없다.
- 테스트의 단위는 동작의 단위이기에 여러 결과가 나올 수 있으며 모든 결과를 평가하는 것이 좋다.
- 다만 검증 구절이 너무 커지는 것은 추상화가 누락되었다는 의미를 가짐
- SUT에서 반환된 객체 내의 모든 속성을 검증하는 대신 동등 멤버를 정의하여 검증(Object equals 메소드)하는 것이 하나의 방법.

### 3.1.6 종료 단계

- 데이터베이스 연결 종료나 작성된 파일 삭제 등이 종료 단계의 예시
- 일반적으로 별도의 메서드로 도출돼 클래스 내 모든 테스트에서 재사용
- 단위 테스트는 프로세스 외부에 종속적이지 않으므로 보통 종료 구절이 필요없음

### 3.1.7 테스트 대상 시스템 구별하기

- SUT를 의존성과 구분하는 것이 중요하며 테스트 내에서 SUT 이름을 sut로 지정하는 것이 도움이 될 수 있음

```java
	public void sumOfTwoNumbers(){
		// 준비
		double first = 10;
		double second = 20;
		var sut = new Calculator();
	
		// 실행
		double result = sut.sum(first, second);

		// 검증
		Assert.Equal(30, result);
	}
```

### 3.1.8 준비, 실행, 검증 주석 제거하기

- AAA 패턴을 따르고 준비 및 검증 구절에 빈 줄을 추가하지 않아도 되는 테스트라면 구절 주석을 제거
- 그렇지 않으면 구절 주석 유지

## 3.3 테스트 간 텍스트 픽스처 재사용

- 테스트 픽스처를 통해 코드를 언제 어떻게 재사용하는지 아는 것은 중요함
- 재사용 방법에 두 가지가 있는데 하나는 유용하고, 다른 하나는 유지 비용을 증가시킴

### 예제1. 테스트 생성자에서 테스트 픽스처를 초기화 하는 방법(비추천)

```java
public class CusdtomerTests{
	private Store store; // 공통 테스트 픽스처
	private Customer sut;

	publicv CustomerTests(){
		store= new Store();
		store.addInventory(Product.Shampoo, 10);
		sut = new Customer();
	}

	public void purchaseSucceedsWhenEnoughInventory(){
		bool success= sut.purchase(store, Product.Shampoo, 5);
		
		Assert.True(success);
	}
	
	public void purchaseFailsWhenNotEnoughInventory(){
		bool success= sut.purchase(store, Product.Shampoo, 15);
		
		Assert.False(success):
	}
}
```

- 위의 예제로 코드의 양을 크게 줄일 수 있으나 다음과 같은 문제가 발생한다

### 단점1. 테스트 간의 높은 결합도는 안티 패턴이다

- 예제에서 모든 테스트는 서로 결합되어 있기에 준비 로직을 수정하면 클래스의 모든 테스트에 영향을 미침(store의 샴푸의 개수가 증가할 경우)
- 이는 테스트를 수정해도 다른 테스트에 영향을 주어서는 안된다는 **격리 지침을 위반**

### 단점2. 테스트 가독성을 떨어뜨리는 생성자 사용

- 테스트만 보고 더 이상 전체 그림을 알 수 없다. 테스트 메서드가  무엇을 하려는지 이해하려면 다른 부분도 봐야함

### 예제2. 테스트 클래스에 비공개 팩토리 메서드를 두는 방법(추천)

```java
public class CusdtomerTests{

	public void purchaseSucceedsWhenEnoughInventory(){
		Store store = createStoreWithInventory(Product.Shampoo, 10);
		Customer sut = createCustomer();

		bool success= sut.purchase(store, Product.Shampoo, 5);
		
		Assert.True(success);
	}
	
	public void purchaseFailsWhenNotEnoughInventory(){
		Store store = createStoreWithInventory(Product.Shampoo, 10);
		Customer sut = createCustomer();

		bool success= sut.purchase(store, Product.Shampoo, 15);
		
		Assert.False(success):
	}

	private Store createStoreWithInventory(Product product, int quantity){
		Store store = new Store();
		store.addInventory(product, quantity);
		return store;
	}

	private static Customer createCustomer(){
		return new Customer();
	}
}
```

- 공통 코드를 추출해 테스트 코드를 짧게 하면서 테스트 진행 상황에 대한 맥락을 유지시킴
- 테스트 간의 결합도도 제거

### 테스트 픽스처 재사용 규칙 예외

- 테스트 전부 또는 대부분에 사용되는(데이터베이스 연결과 같은) 경우 생성자에서 픽스처를 초기화할 수 있다
- 하지만 이 역시 개별 클래스 생성자보다 기초 클래스를 두어 상속을 통해 사용하는 것이 더 합리적

## 3.4 단위 테스트 명명법

- 개발자 최대의 적 이름 짓기
- [테스트 대상 메서드]_[시나리오]_[예상 결과] 같이 오래된 명명 관습이 있지만 별로 도움이 되지 않음
- 쉬운 구문이 더 효과적일 수 있다(Sum_of_two_numbers 가 Sum_TwoNumbers_ReturnSum 보다 낫다)

### 3.4.1 단위 테스트 명명 지침

- 엄격한 명명 정책을 따르지 않는다
- 문제 도메인에 익숙한 비개발자들에게 시나리오를 설명하는 것처럼 이름을 짓자
- 단어를 밑줄로 구분한다. 그러면 긴 이름에서 가독성이 향상된다(클래스 명은 길지 않아서 camelCase여도 상관없지만 테스트 메소드 이름은 길기에 snake_case)

### 3.4.2 예제: 지침에 따른 테스트 이름 변경

- IsDeliveryValid_InvalidDate_ReturnFalse 와 같은 메서드를 다음과 같이 변경할 수 있음
- delivery_with_invalid_date_should_be_considered_invalid → 여기서 invalid date라는 표현을 좀 더 알기 쉽게 바꾸자면
- delivery_with_past_date_should_be_considered_invalid → 너무 장황함
- delivery_with_past_date_should_be_invalid → sholud be는 단순하고 원자적인 사실이 아닌 소망, 욕구이기에 부적절
- delivery_with_past_date_is_invalid → 기초 영문법만 지킨다면
- 최종적으로 delivery_with_a_past_date_is_invalid 이해하기 쉬운 문장이 나옴
- 추가적으로 테스트 이름에 SUT의 메서드 이름을 포함하지 말아야 한다. 코드를 테스트 하는 것이 아닌 **동작**을 테스트 하는 것이고 SUT는 단지 진입점, 동작을 호출하는 수단일 뿐이다. 예외는 비즈니스 로직이 없는 유틸리티 코드