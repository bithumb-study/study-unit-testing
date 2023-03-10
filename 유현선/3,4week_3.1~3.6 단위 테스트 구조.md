# 3장_단위 테스트 구조

## 3.1 단위 테스트를 구성하는 방법

### 3.1.1 AAA 패턴 사용

코드 예시
```java

// 테스트 대상 코드
public class Calculator {
    public int sum(int first, int second) {
        return first + second;
    }
}


// 테스트 코드
public class CalculatorTests {
    
    @Test
    void sum_of_two_numbers() {
        // 준비 (Arrange)
        int first = 10;
        int second = 20;
        Calculator calculator = new Calculator();
        
        // 실행 (Act)
        int result = calculator.sum(first, second);
        
        // 검증 (Assert) 
        assertEquals(30, result);
    }
}
```

AAA 패턴의 장점
- 일관성 : 스위트 내 모든 테스트가 단순하고 균일한 구조를 갖는데 도움이 됨
- 익숙해지면 모든 테스트를 쉽게 읽을 수 있고 이해할 수 있음 -> 테스트 스위트의 유지 보수 비용이 줄어듬

AAA 패턴 구조
- 준비 (Arrange) : 테스트 대상 시스템(SUT, System Under Test) 과 해당 의존성을 원하는 상태로 만듦
- 실행 (Act) : SUT 에서 메서드를 호출하고 준비된 의존성을 전달하며 (있다면) 출력 값을 캡쳐
- 검증 (Assert) : 결과를 검증. 결과는 반환 값이나 SUT 와 협력자의 최종 상태, SUT 가 협력자에 호출한 메서드 등으로 표시될 수 있다. 

AAA 패턴과 Given-When-Then 패턴
- Given = Arrange
- When = Act
- Then = Assert
- 차이점 : Given-When-Then 구조가 비기술자들과 공유하는 데 더 읽기 쉬움



### 3.1.2 여러 개의 준비, 실행, 검증 구절 피하기
준비, 실행, 검증 구절이 여러개로 중첩되는 경우 = 테스트가 너무 많은 것을 한 번에 검증
:: 해결 - 테스트를 여러개로 나눠서 작성한다.
(통합 테스트에서는 실행 구절을 여러 개 두는 것이 괜찮을 때도 있다.)


### 3.1.3 테스트 내 if 문 피하기 
테스트 내의 if 문은 일종의 안티 패턴으로 if 문이 있는 경우라면 반드시 여러 테스트로 나눠야한다.


### 3.1.4 각 구절은 얼마나 커야 하는가? 
- 준비 : 제일 큼
  - 코드 재사용에 도움이 되는 패턴 : 오브젝트 마더 / 테스트 데이터 빌더
- 실행 : 보통 코드 한 줄 
  - 2줄 이상일 경우 
    - SUT의 공개 API에 문제가 있다는 의미 
    - 불변 위반 
    - 해결책 : 코드 캡슐화를 통해 잠재적 모순으로부터 코드를 보호 



### 3.1.5 검증 구절에는 검증문이 얼마나 있어야 하는가 
- 단일 동작 단위는 여러 결과를 낼 수 있으며, 하나의 테스트로 그 모든 결과를 평가하는 것이 좋음
- But, 검증 구절이 너무 커지는 것은 경계해야함
- 객체 클래스 내에 적절한 동등 맴버를 정의하는 것이 좋음 : 단일 검증문으로 객체를 기대값과 비교 



### 3.1.6 종료 단계
- AAA 패턴, 대부분의 단위 테스트는 종료 구절이 필요없음
- 종료 구절은 통합 테스트의 영역


### 3.1.7 테스트 대상 시스템 구별하기
- SUT : 애플리케이션에서 호출하고자 하는 동작에 대한 진입점을 제공 (오직 하나만 존재)
- SUT 를 의존성과 구분하자 
-  ```Calculator calculator = new Calculator(); -> Calculator sut = new Calculator();```


### 3.1.8 준비, 실행, 검증 주석 제거하기
- AAA 패턴을 따르고 준비 및 검증 구절에 빈 줄을 추가하지 않아도 되는 테스트라면 구절 주석 지우기 

---

## 3.2 xUnit 테스트 프레임워크 살펴보기 
### JUnit
- https://junit.org/junit5/

---

## 3.3 테스트 간 테스트 픽스처 재사용 

테스트 픽스처 
- 테스트 실행 대상 객체 
- 각 테스트 실행 전에 알려진 고정 상태로 유지하기 때문에 동일한 결과를 생성

```java

public class CustomerTests {
    private Store store;
    private Customer sut;
    
    @BeforeEach
    void startTest() {
        store = new Store();
        store.addInventory(Product.Shampoo, 10);
        sut = new Customer();
    }
    
    @Test
    void purchase_succeeds_when_enough_inventory() {
        boolean success = sut.purchase(store, Product.Shampoo, 5);
        
        assertTrue(success);
        assertEquals(5, store.getInventory(Product.Shampoo));
    }
    
    @Test
    void purchase_fails_when_not_enough_inventory() {
        boolean success = sut.purchase(store, Product.Shampoo, 15);

        assertFalse(success);
        assertEquals(10, store.getInventory(Product.Shampoo));
    }
}

```

- 테스트 간 결합도가 높아짐
- 테스트 가독성이 떨어진다.


### 3.3.1 테스트 간의 높은 결합도는 안티 패턴이다
- 테스트를 수정해도 다른 테스트에 영향을 주어서는 안됨 
- 테스트 클래스에 공유 상태를 두면 안됨

### 3.3.2 테스트 가독성을 떨어뜨리는 생성자 사용
- 테스트만 보고는 전체 그림을 볼 수 없는 문제가 생김
- 독립적인 테스트를 작성하여 불확실성을 없애자.

### 3.3.3 더 나은 테스트 픽스처 재사용법
```java 
public class CustomerTests {
    @Test
    void purchase_succeeds_when_enough_inventory() {
        Store store = createStoreWithInventory(Product.Shampoo, 10);
        Customer sut = createCustomer();
        
        boolean success = sut.purchase(store, Product.Shampoo, 5);
        
        assertTrue(success);
        assertEquals(5, store.getInventory(Product.Shampoo));
    }
    
    @Test
    void purchase_fails_when_not_enough_inventory() {
        Store store = createStoreWithInventory(Product.Shampoo, 10);
        Customer sut = CreateCustomer();

        boolean success = sut.purchase(store, Product.Shampoo, 15);

        assertFalse(success);
        assertEquals(10, store.getInventory(Product.Shampoo));
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


---

## 3.4 단위 테스트 명명법 

가장 유명한 방법 : `[테스트 대상 메서드]_[시나리오]_[예상 결과]`
- ex) `Sum_TwoNumbers_ReturnsSum()` vs `Sum_of_two_numbers()`
- 그러나 이 방법은 전혀 도움이 되지 않는다. (가독성 X, 의미 전달이 바로 되지 않음)
- 쉬운 영어로 작성한 이름이 읽기에 훨씬 간결하고 테스트 대상 동작에 대한 현실적인 설명이다.


### 3.4.1 단위 테스트 명명 지침
- 엄격한 명명 정책을 따르지 않는다. 표현의 자유롤 허용
- 문제 도메인에 익숙한 비개발자들에게 시나리오를 설명하는 것처럼 테스트 명명. 도메인 전문가나 비즈니스 분석가가 좋은 예시
- 단어를 언더스코어(_) 로 구분

코드를 테스트 하는 것이 아니라 어플리케이션 동작을 테스트하는 것임을 명심하는게 중요  


---

## 3.5 매개변수화된 테스트 리팩터링하기 

> JUnit : Parameterized 

---

## 3.6 검증문 라이브러리를 사용한 테스트 가독성 향상

> AssertJ  
> http://joel-costigliola.github.io/assertj/  
