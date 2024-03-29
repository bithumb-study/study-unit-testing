# [Unit Testing] 17장 단위 테스트 안티 패턴

## 11.1 비공개 메서드 단위 테스트

- 비공개 메서드 테스트는 "전혀 하지 말아야 한다."

### 11.1.1 비공개 메서드와 테스트 취약성

- 비공개 메서드를 노출하는 경우는 식별할 수 있는 동작만 테스트하는 것을 위반한다.
- 노출 하게되면 구현 세부사항과 결합 리팩터링 내성이 떨어진다.
- 포괄적으로 식별할수 있는 동작과 간접적으로 테스트 하는것이 좋다.

### 11.1.2 비공개 메서드와 불필요한 커버리지

- 죽은코드 , 테스트에서 벗어난 코드가 어디에도 사용되지 않는다면 리팩터링 후에도 남아서 관계없는 코드 일수있다. (삭제가 좋다.)
- 추상화가 누락돼어 복잡한 비공개 메서드를 별도의 클래스로 추상화가 누락된 징후

### 11.1.3 비공개 메서드 테스트가 타당한 경우

- 식별할 수 있는 동작을 공개로 하고 구현 세부사항을 비공개로 하면 잘 설계된 API 반면 구현 세부사항이 노출되면 코드 캡슐화를 해친다.
- 메서드가 비공개 이면서 식별할 수있는 동작이면 테스트해도 나쁘지 않지만 드물다.

## 11.2 비공개 상태 노출

- 단위 테스트 목적으로만 비공개 상태를 노출하는것 (안티패턴)

## 11.3 테스트로 유출된 도메인 지식

- 도메인 지식을 테스트로 유출하는것 또한 안티패턴
- 제품코드의 알고리즘 구현을 복사 한것  이러한 테스트는 구현세부 사항과 결합되는 또 다른 예
  - 리팩터링 내성이 떨어지고 거짓양성과 타당한 실패를 구별할수가 없게 된다.
- 알고리즘을 복사하는 대신 예상결과를 하드코딩하는 것이 좋다.

## 11.4 코드 오염
#### 코드 오염은 테스트에만 필요한 제품 코드를 추가 하는것이다.

- 이렇게 되면 테스트 코드와 제품 코드가 혼재돼 유지비가 증가 
- 방지하기 위해서는 테스트 코드를 제품 코드베이스와 분리 해야 한다.

## 11.5 구체 클래스를 목으로 처리하기

- 구체 클래스를 대신 목으로 처리해서 본체 클래스의 기능 일부를 보존할수 있지만 이는 단일 책임 원칙에 위배 하는 중대한 단점이 된다.

## 11.6 시간 처리하기

- 시간에 따라 달라지는 기능을 테스트하면 거짓양성이 발생할수 있다.
- 세가지 방법중 하나는 안티패턴 두개는 바람직한 방법

### 11.6.1 엠비언트 컨텍스트로서의 시간

- 안티패턴의 하나 
- 제품 코드를 오염시키고 테스트를 어렵게 만든다. 
- 정적 필드는 테스트 간에 공유되는 의존성을 도입해 해당테스트를 통합 테스트 영역으로 전환

### 11.6.2 명시적 의존성으로의 시간

- 서비스 또는 일반 값으로 시간 의존성을 명시적으로 주입하는것 
- 시간을 서비스로 주입하는 것이 더 낫다.
- 제품 코드에서 일반 값으로 작업하는 것이 더 쉽고 테스트에서 해당 값을 스텁으로 처리하기도 쉽다.


