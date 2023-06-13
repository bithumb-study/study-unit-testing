# Unit Testing : 생산성과 품질을 위한 단위 테스트 원칙과 패턴

이 내용은 [단위 테스트 생산성과 품질을 위한 단위 테스트 원칙과 패턴]을 읽으면서 정리한 내용을 포함하고 있습니다.

- 11장 단위 테스트 안티 패턴 : 11.1 ~ 11.7

목차는 다음과 같습니다.

- 11.1 비공개 메서드 단위 테스트
- 11.2 비공개 상태 노출
- 11.3 테스트로 유출된 도메인 지식
- 11.4 코드 오염
- 11.5
- 11.6
- 11.7 

## 11장 단위 테스트 안티 패턴

안티 패턴은 겉으로 적절한 것처럼 보이지만 장래에 더 큰 문제로 이어지는 반복적인 문제에 대한 일반적인 해결책이다.
여기서는 테스트에서 시간을 어떻게 다루는지 알아보고, 비공개 메서드 단위 테스트, 코드 오염, 구체 클래스 목 처리 등과 같은 안티 패턴을 설명하여 이를 피하는 방법도 살펴본다.

### 11.1 비공개 메서드 단위 테스트

비공개 메서드를 어떻게 테스트해야 할까? 라는 질문의 대한 대답은 무엇일까?

#### 11.1.1 비공개 메서드와 테스트 취약성

단위 테스트를 하려고 비공개 메서드를 노출하게 되면 식별할 수 있는 동작만 테스트하는 것을 위반하게 된다.
비공개 메서드를 노출하면 테스트가 구현 세부 사항과 결합되고 결과적으로 리팩터링 내성이 떨어진다.
비공개 메서드를 직접 테스트하는 대신, 포괄적인 식별할 수 있는 동작으로서 간접적으로 테스트하는 것이 좋다.

#### 11.1.2 비공개 메서드와 불필요한 커버리지

비공개 메서드가 너무 복잡해서 식별할 수 있는 동작으로 테스트하기에 충분히 커버리지를 얻을 수 없는 경우

- 죽은 코드일 확률
- 추상화가 누락되어 있을 확률

추상화가 누락되어 있을 확률의 예를 들어보면 다음과 같다.

```java
public class Order {

    private Customer customer;
    private List<Product> products;

    public String generateDescription() {
        return "Customer name : " + customer.getName() +
                "Total number of products : " + products.getCount() +
                "Total prict : " + getPrice();
    }
}

- 복잡한 비공개 메서드인 `getPrice()` 가 있는 경우

```java
public class Order {
    
    private Customer customer;
    private List<Product> products;

    public String generateDescription() {

        var calc = new PriceCalculator(); 

        return "Customer name : " + customer.getName() +
                "Total number of products : " + products.getCount() +
                "Total prict : " + calc.calculate(customer, products);
    }
}
```

```java
public class PriceCalculator {

    public BigDecimal calculate(final Customer customer, final List<Product> products) {
        BigDecimal basePrice = /* products에 기반한 계산 */;
        BigDecimal discounts = /* customer에 기반한 계산 */;
        BigDecimal taxes = /* products에 기반한 계산 */;
        return basePrice.subtract(discounts).add(taxes)
    }
}
```

- `Order`와 별개로 `PriceCalculator`를 테스트 가능

#### 11.1.3 비공개 메서드 테스트가 타당한 경우

비공개 메서드를 절대 테스트하지 말라는 규칙에도 예외는 있다.

|    | 식별할 수 있는 동작 | 구현 세부 사항 |
|----|-----------------|------------|
|공개 | 좋음             | 나쁨        |
|비공개|해당 없음         | 좋음         |

- 식별할 수 있는 동작을 공개로 하고 구현 세부 사항을 비공개로 하면 API가 잘 설계되었다고 할 수 있다.
- 반면에 구현 세부 사항이 유출되면 코드 캡슐화를 해치게 된다.
- 메서드가 식별할 수 있는 동작이 되려면 클라이언트 코드에서 사용돼야 하므로 해당 메서드가 비공개인 경우에는 불가능
- 비공개 메서드를 테스트하는 것 자체는 나쁘지 않으나 비공개 메서드가 구현 세부 사항의 해당하므로 구현 세부 사항을 테스트하면 궁극적으로 테스트가 깨지기 쉽다.

구현 세부 사항의 해당하지 않으며 공개 해도 테스트가 쉽게 깨지지 않는 경우에는 공개해서 테스트를 진행해도 상관없다.
또한 클래스의 공개 API 노출 영역을 가능한 한 작게 하려면 테스트에서 리플렉션을 활용할 수 있다.

### 11.2 비공개 상태 노출

테스트를 위한 목적으로 상태 변경의 활용하는 것은 안티 패턴이다.
테스트는 제품 코드와 정확히 같은 방식으로 테스트 대상 시스템과 상호 작용해야 하며, 특별한 권한이 따로 있어서는 안 된다.

```
테스트 유의성을 위해 공개 API 노출 영역을 넓히는 것은 좋지 않은 관습이다.
```

### 11.3 테스트로 유출된 도메인 지식

도메인 지식을 테스트로 유출하는 것은 또 하나의 흔한 안티 패턴이며, 보통 복잡한 알고리즘(또는 비즈니스 로직)을 다루는 테스트에서 일어난다.

```java
public static class Calculator {
    public static int Add(final int value1, final int value2) {
        return value1 + value2;
    }
}
```

다음은 잘못된 테스트 방법이다.

```java
public class CalcularotTests {

    @Test
    void adding_two_numbers() {
        
        int value1 = 1;
        int value2 = 3;
        int expected = value1 + value2; // 유출

        int actual = calculator.add(value1, value2);

        assertThat(actual).isEqual(expected);
    }
}
```

- 안티 패턴의 예
- 테스트를 작성 때 특정 구현을 암시하면 안 된다.

다음과 같이 변경할 수 있다.

```java
public class CalcularotTests {

    @Test
    void adding_two_numbers() {
        
        int value1 = 1;
        int value2 = 3;
        int actual = calculator.add(value1, value2);

        assertThat(actual).isEqual(value1 + value2);
    }
}
```

- 단위 테스트에서는 예상 결과를 하드코딩하는 것이 좋다.

테스트를 작성할 때 특정 구현을 암시하면 안 된다.

- 블랙박스 관점에서 제품 코드를 검증해야 한다.
- 도메인 지식을 테스트에 유출하지 않도록 해야 한다.

### 11.4 코드 오염

다음 안티 패턴은 코드 오염이다.

- 코드 오염은 테스트에만 필요한 제품 코드를 추가하는 것이다.

코드 오염의 문제는 테스트 코드와 제품 코드가 혼재돼 유지비가 증가하는 것이다. 이러한 안티 패턴을 방지하려면 테스트 코드를 제품 코드 베이스와 분리해야 한다.

```java
public interfae ILogger {
    void log(final String text);
}

public class Logger implements ILogger {
    public void log(final String text) {
        /* text에 대한 로깅 */
    }
}

public class FakeLogger implements ILogger {
    public void log(final String text) {
        /* 아무것도 하지 않음 */
    }
}

public class Controller {
    public void someMethod(final ILogger logger) {
        logger.log("SomeMethod 호출");
    }
}
```

- 이렇게 분리하면 더 이상 다른 환경을 설명할 필요가 없으므로 단순하게 할 수 있다.

### 11.5 구체 클래스를 목으로 처리하기

목으로 처리하는 방법이외에도 한 가지 방법이 더 있다.

- 구체 클래스를 대신 목으로 처리해서 본래 클래스의 기능 일부를 보존할 수 있다.

기능 지키려고 구체 클래스를 목으로 처리해야 하면, 이는 단일 책임 원칙을 위반하는 결과이다.

- 해당 클래스를 두 가지 클래스, 즉 도메인 로직이 있는 클래스와 프로세스 외부 의존성과 통신하는 클래스로 분리

### 11.6 시간 처리하기

시간에 따라 달라지는 기능을 테스트하면 거짓 양성이 발생할 수 있다.

#### 11.6.1 앰비언트 컨텍스트로서의 시간

내장된 함수를 사용하는 것이 아닌 다음 예제와 같이 코드에서 사용할 수 있는 사용자 정의 클래스를 사용하는 것이다.

- Date 또는 LocalDate

#### 11.6.2 명시적 의존성으로서의 시간

더 나은 방법으로 서비스 또는 일반 값으로 시간 의존성을 명시적으로 주입하는 것이 있다.

### 11.7 결론

실전 단위 테스트 사용 사례 및 좋은 테스트의 4대 요소를 이용한 사례 분석