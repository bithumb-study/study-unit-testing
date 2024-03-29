# [Unit Testing] 9장 목 처리에 대한 모범  사례

- 목은 테스트 대상 시슽메과 의존성 간의 상호 작용을 모방하고 검사 하는데 도움이 되는 테스트 대역
- 비관리 의존성에만 적용해야 한다.

## 9.1 목의 가치를 극대화 하기

- 비관리 의존성에만 목을 사용

### 9.1.1 시스템 끝에서 상호 작용 검증하기

- 시스템 끝에서 비관리 의존성과의 상호 작용을 검증하라
  - 회귀방지 향상 및 리팩터링 내성 향상
  - 코드베이스와의 결합도 가 낮기 때문에 낮은 수준의 리팩터링에도 영향을 많이 받지않는다.
- 비관리 의존성과 통신하는 마지막 타입을 목으로 처리하면 통합 테스트가 거치는 클래스의 수가 증가하여 보호가 향상

### 9.1.2 목을 스파이로 대체하기

- 스파이는 목과 같은 목적을 수행하는 테스트대역 (스파이는 수동 작성 , 목은 프레임워크 도움)
- 스파이는 검증 단계에서 코드를 재사용해 테스트 크기를 줄이고 가독성을 향상

## 9.2 목 처리에 대한 모범 사례

- 비관리 의존성에만 목 적용
- 시스템 끝에 있는 의존성에 대해 상호작용 검증
- 통합 테스트에서만 목을 사용하고 단위테스트에서는 하지 않기
- 항상 목 호출 수 확인
- 보유 타입만 목으로 처리

### 9.2.1 목은 통합 테스트만을 위한 것

- 단위테스트에서는 목을 사용하면 안된다.
- 도메인 모델에 대한 테스트는 단위 테스트 범주에 속하므로 목을 사용하지 않는다.
- 컨트롤러를 다루는 테스트는 통합테스트
  - 목은 비관리 의존성을 다루기 때문에 컨트롤러만 이러한 의존성을 처리하는 코드이기 때문에 목을 적용

### 9.2.2 테스트당 목이 하나일 필요는 없음

- 동작 단위를 검증하는 데 필요한 목의 수는 관계가 없다.
- 통합테스트에 사용할 목의 수를 통제 할수 없다.
- 오직 운영에 참여하는 비관리 의존성 수에만 의존한다.

### 9.2.3 호출 횟수 검증하기

- 비관리 의존성과의 통신에 관해서는 다음 두 가지 모두 확인하는것이 중요
  - 예상하는 호출이 있는가?
  - 예상치 못하는 호출이 있는가?
- 비관리 의존성과의 하위 호환성을 지켜야 하는데서 비롯 호환성은 양방향이어야 한다.

### 9.2.4 보유 타입만 목으로 처리하기

- 서드파티 라이브러리 위에 항상 어댑터를 작성하고 기본 타입 대신 해당 어댑터를 목으로 처리해야한다.
  - 서드파티 코드의 작동 방식에 대해 깊이 이해하지 못한경우가 많다.
  - 서드파티가 내장 인터페이스를 제공하더라도 목으로 처리한 동작이 실제로 외부 라이브러리와 일치하는지 확인해야하므로 목으로 처리하는건 위험하다.
- 어댑터는 코드와 외부 환경 사이의 손상 방지 계층으로 작동
  - 기본 라이브러리의 복잡성을 추상화
  - 라이브러리에 필요한 기능만 노출
  - 프로젝트 도메인 언어를 사용해 수행할수 있다.
