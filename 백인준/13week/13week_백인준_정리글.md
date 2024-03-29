# [Unit Testing] 8장 통합 테스트를 하는 이유

## 8.5 통합 테스트 모범 사례
- 도메인 모델 경계 명시하기
- 애플리케이션 내 계층 줄이기
- 순환 의존성 제거하기

### 8.5.1 도메인 모델 경계 명시하기
- 항상 도메인 모델을 코드베이스 에서 명시적이고 잘알려진 위치에 두도록 하라
  - 코드의 해당부분을 더 잘보여주고 잘 설명할수있다.
  - 단위 테스트와 통합 테스트의 차이점을 쉽게 구별
  - 별도의 어셈블리 또는 네임 스페이스 형태를 취할수 있다,.

### 8.5.2 계층 수 줄이기
- 애플리케이션에 추상 계층이 많으면 코드베이스를 탐색하기 어렵고 숨은 로직을 이해하기 어렵다.
  - 단위 테스트와 통합 테스트 에도 도움이 되지 않는다.
- 간접계층은 코드를 추론하는데 부정적영향
  - 간접계층이 많은 코드는 컨트롤러와 도메인 모델 사이에 명확한 경계가 없는편
  - 통합 테스트 가치가 떨어진다.
- 가능한 간접계층을 적게 사용하라

### 8.5.3 순환 의존성 제거하기
- 순환 의존성 대표는 콜백
- 추상계층과 마찬가지로 순환의존성은 코드를 읽고 이해하려 할때 알아야 할것이 많아 부담스럽다.
- 순환 의존성은 테스트를 방해
  - 클래스 그래프를 나눠서 동작단위를 하나 분리하려면 목으로 처리해야 하므로 

### 8.5.4 테스트에서 다중 실행 구절 사용
- 두개이상의 준비,실행,검증은 코드악취 
- 각 실행을 고유의 테스트로 추출해 테스트를 나누는것이 좋다.
  - 단일 동작 단위에 초점을 맞추게 되면 테스트를 더 쉽게 이해하고 필요 할때 수정가능
- 둘 이상의 실행구절을 사용하는 경우는 외부 의존성을 관리하기 어려운 경우뿐

## 8.6 로깅 기능을 테스트 하는 방법
- 로깅을 조금이라도 테스트 해야하는가?
- 만약 그렇다면 어떻게 테스트해야 하는가?
- 로깅이 얼마나 많으면 충분한가?
- 로거 인스턴스를 어떻게 전달할까?

### 8.6.1 로깅을 테스트 해야하는가?
- 로깅은 횡단 기능으로 코드 어느부분에서나 필요할수 있다.
- 로깅은 애플리케이션의 동작에 대해 중요한 정보를 생성 
  - 그러나 로깅은 너무나 보편적이므로 테스트 노력을 더 들일 가치가 있는지 분명하지 않다.
- 보는이가 개발자 뿐이라면 구현 세부 사항이기 때문에 테스트해서는 안된다.
- 로깅 유형
  - 지원 로깅: 지원 담당자 , 시스템관리자가 추적 할수 있는 메시지 생성
  - 진단 로깅: 개발자가 애플리케이션 내부 상황을 파악할수 있도록 돕는다.

### 8.6.2 로깅을 어떻게 테스트 해야 하는가?
- 로깅에는 프로세스 외부 의존성이 있기 때문에 테스트에 관한 한 프로세스 외부 의존성에 영향을 주는 다른 기능과 동일한 규칙 적용

### 8.6.3  로깅이 얼마나 많으면 충분한가?
- 지원 로깅은 비즈니스 요구 사항이기 떄문에 제외
- 진단로깅은 조절 (과도하게 사용하지 않는다.)
  - 도메인 모델에서 특히 과도한 로깅은 코드를 혼란스럽게 한다. 코드가 모호 해진다.
  - 로그의 신호 대비 잡음 비율
    - 신호를 최대한 늘리고 잡음을 최소한으로 줄여라

### 8.6.4 로거 인스턴스를 어떻게 전달하는가?
- 정적메서드를 사용
  - 안티 패턴 
    - 의존성이 숨어있고 변경하기 어렵다.
    - 테스트가 더 어려워 진다.
- 클래스 생성자를 통해 전달

## 8.7 결론
- 식별할 수있는 동작인지 아니면 구현 세부 사항인지 여부에 대한 관점으로 프로세스 외부 의존성과의 통신을 살펴보자
- 개발자가 아닌 사람이 로그를 볼수있으면 목으로 처리 아니면 테스트 하지 말라