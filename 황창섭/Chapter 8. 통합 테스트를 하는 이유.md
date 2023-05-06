# Chapter 8. 통합 테스트를 하는 이유

# Chapter 8. 통합 테스트를 하는 이유

- 단위 테스트로만으로는 시스템이 전체적으로 잘 잘동하는지 확신할 수 없음
- 해당 챕터에서 학습하는 내용
    - 통합 테스트의 역할
    - 프로세스 외부 의존성 중에서 어느 것을 통합 테스트에서 그대로 사용하고 어느 것을 목으로 대체할지 학습
    - 코드베이스 상태를 개선하는 데 도움이 되는 모범 통합 테스트 사례

## 8.1 통합 테스트란 무엇인가?

### 8.1.1 통합 테스트의 역할

- 단위 테스트의 요구 사항
    - 단일 동작 단위를 검증하고
    - 빠르게 수행하고
    - 다른 테스트와 별도로 처리
- 이 세 가지 중 하나라도 충족하지 못하는 테스트는 통합 테스트 범주에 속함

### 8.1.2 다시 보는 테스트 피라미드

- 단위 테스트로 가능한 한 많은 비즈니스 시나리오의 예외 상황을 확인하고, 통합 테스트는 주요 흐름과 단위 테스트가 다루지 못하는 기타 예외 상황을 다룸
- 프로그램의 복잡도가 높을 수록 단위 테스트의 양이 많으며, 일반적으로 단위 테스트 양이 통합 테스트 양보다 많은 피라미드 형태

### 8.1.3 통합 테스트와 빠른 실패

- 통합 테스트는 가장 긴 주요 흐름(happy path)를 선택한다
- 모든 상호 작용을 거치는 흐름이 없으면 외부 시스템과의 통신을 모두 확인하는 데 필요한 만큼 통합 테스트를 추가로 작성한다.
- 빠른 실패 원칙
    - 예기치 않은 오류가 발생하자마자 현재 연산을 중단하는 것을 의미
    - 보통 예외를 던져서 현재 연산을 중지함
    - 이 원칙으로 애플리케이션의 안정성을 높임

## 8.2 어떤 프로세스 외부 의존성을 직접 테스트해야 하는가?

- 통합 테스트는 시스템이 프로세스 외부 의존성과 어떻게 통합하는지를 검증
- 이는 실제 외부 의존성을 사용하거나 목으로 대체
- 해당 절에서 두 가지 방식을 어떤 상황에서 선택해 적용하는지 학습

### 8.2.1 프로세스 외부 의존성의 두 가지 유형

- 프로제스 외부 의존성의 두 가지 범주
    - 관리 의존성(전체를 제어할 수 있는 프로세스 외부 의존성)
        - 애플리케이션을 통해서만 접근하며 해당 의존성과의 상호 작용은 외부 환경에서 볼 수 없음
        - 대표적인 예로 데이터베이스
        - 구현 세부 사항에 속함
        - 실제 인스턴스를 사용하여 통합 테스트 작성
    - 비관리 의존성
        - 해당 의존성과의 상호 작용을 외부에서 볼 수 있음
        - SMTP 서버와 메시지 버스 등
        - 식별할 수 있는 동작에 속함
        - 목으로 대체하여 통합 테스트 작성

### 8.2.2 관리 의존성이면서 비관리 의존성인 프로세스 외부 의존성 다루기

- 관리 의존성과 비관리 의존성 모두의 속성을 나타내는 외부 의존성도 있다.(예: 다른 애플리케이션에서 접근할 수 있는 데이터베이스)
- 이는 피하는 것이 좋으나 어쩔 수 없이 사용하게 된다면 다른 애플리케이션에서 볼 수 있는 테이블을 **비관리 의존성**으로 취급한다
- 이러한 테이블은 사실상 메시지 버스 역할을 하고 각 행이 메시지 역할을 함
- 다른 애플리케이션에서 볼 수 있는 테이블은 목으로 대체하고 직접 관리하는 테스트는 직접 테스트 함

### 8.2.3 통합 테스트에서 실제 데이터베이스를 사용할 수 없으면 어떻게 할까?

- IT 보안 정책 등의 이유로 통합 테스트에서 관리 의존성을 실제 버전으로 사용할 수 없는 경우도 있음
- 이러한 경우에 테스트를 위해 억지로 목으로 대체한다면 리팩터링 내성이 저하되고 회귀 방지도 떨어지게 되는 결과가 초래된다
- 통합 테스트의 가치가 떨어지기에 아예 작성하지 않고 도메인 모델의 단위 테스트에만 집중!(**가치가 떨어지는 테스트는 테스트 스위트에 있어서는 안된다**)

### 8.3 통합 테스트: 예제

```java
public class UserController {
    private Database database = new Database();

    private MessageBus messageBus = new MessageBus();

    public String changeEmail(int userId, String newEmail) {
        // 데이터베이스에서 사용자의 현재 이메일과 유형 검색
        Object[] userData = Database.getUserById(userId);
        User user = UserFactory.create(data);
        
        String error = user.canChangeEmail();
        if(error != null)
            return error;
        
        // 데이터베이스에서 조직의 도메인 이름과 직원 수 검색
        Object[] companyData = database.getCompany();
        Company company = CompanyFactory.create(companyData);
        
        user.ChangeEmail(newEmail, companyDomainName, numberOfEmployees);

        // 데이터베이스에 사용자 저장
        database.SaveCompany(newNumberOfEmployees);
        database.saveUser(user);
        
        // 메시지 버스에 알림 전송
        message.sendEmailChangedMessage(userId, newEmail);
        
        return "OK";
    }
}
```

- 사용자의 이메일을 변경하는 예제. 컨트롤러 모습

### 8.3.1 어떤 시나리오를 테스트할까?

- 통합 테스트에 대한 일반적인 지침은 **가장 긴 주요 흐름**과 **단위 테스트로 수행할 수 없는 모든 예외 상황**을 다루는 것
- 위의 코드에서 가장 긴 주요 흐름은 기업 이메일에서 일반 이메일로 변경하는 것
- 위의 흐름에서의 사이드 이펙트
    - 데이터베이스에서 사용자와 회사 모두 업데이트 된다. 사용자는 유형을 변경하고(기업에서 일반으로) 이메일도 변경하며 회사는 직원 수를 변경한다
    - 메시지 버스로 메시지를 보낸다
- 한 가지 예외 상황이 있지만(이메일을 변경할 수 없는 시나리오) 이 시나리오는 빨리 실패하기에 테스트할 필요는 없다

### 8.3.2 데이터베이스와 메시지 버스 분류하기

- 위의 예제에서 데이터베이스는 관리 의존성이고 메시지 버스는 비관리 의존성이기에 목으로 대체한다

### 8.3.3 엔드 투 엔드 테스트는 어떤가?

- 엔드 투 엔드 테스트의 사용 여부는 프로젝트 상황에 따라 판단
- 통합 테스트는 비관리 의존성만 목으로 대체하기에 엔드 투 엔드 테스트를 어느 정도 생략할 수 있음
- 배포 후 프로젝트의 상태 점검을 위해 한 개 또는 두 개 정도의 중요한 엔드 투 엔드 테스트를 작성할 수 있음

### 8.3.2 통합 테스트: 첫 번째 시도

```java
public class UserControllerTest{
    public void changing_email_from_corporate_to_non_corportate(){
        // given
        Database db = new Database(ConnectionString);
        CreateUserToDb("user@mycorp.com", UserType.Employee, db);
        CreateCompanyToDb("mycorp.com", 1, db);

        MessageBus messageBusMock = new Mock<MessageBus>();
        UserController sut = new UserController(db, messageBusMock);

        // when
        String result = sut.changeEmail(user.getUserId(), "new@gmail.com");

        // then
        Assert.Equal("OK", result);

        // User 상태 검증
        Object[] userData = db.getUserById(user.getUserId());
        User userFromDb = UserFactory.Create(userData);
        Assert.Equal("new@gmail.com", userFromDb.getEmail());
        Assert.Equal(UserType.Customer, userFromDb.getType());

        // 회사 상태 검증
        Object[] companyData = db.getCompany();
        Company companyFromDb = CompanyFactory.create(companyData);
        Assert.Equal(0, companyFromDb.getNumberOfEmployees);
        
        // 목 상호 작용 확인
        messageBusMock.Verify(x -> x.sendEmailChangedMessage(user.getUserId(), "new@gmail.com"), Times.Once);
    }
}
```

- 위의 통합 테스트는 개선할 여지가 남아있음
- 예를 들어 헬퍼 메서드를 통해 검증 구절의 크기를 줄일 수 있음
- 그리고 messageBusMock은 회귀 방지가 그다지 좋지 않음

## 8.4 의존성 추상화를 위한 인터페이스 사용

- 단위 테스트 영역에서 가장 남용하는 부분이 인터페이스 사용

### 8.4.1 인터페이스와 느슨한 결합

- 데이터베이스나 메시지 버스와 같은 프로세스 외부 의존성을 위해 인터페이스를 도입(심지어 구현이 하나만 있는 경우에도)
- 왜냐하면 인터페이스를 사용하는 일반적인 이유가
    - 외부 의존성을 추상화해 느슨한 결합을 달성하고
    - 기존 코드를 변경하지 않고 새로운 기능을 추가해 OCP 원칙을 준수하기 떄문
- 하지만 두 가지 이유 모두 오해며 단일 구현을 위한 인터페이스는 추상화가 아니며 결합도 또한 낮지 않음
- 인터페이스가 진정으로 추상화되려면 구현이 적어도 두 가지는 있어야 함
- 또한 현재 필요하지 않는 기능에 시간을 들일 필요가 없다 왜냐하면
    - 기회비용: 비즈니스 담당자들에게 필요하지 않은 기능에 시간을 보낸다면 시간을 허비하는 것. 처음부터 실제 필요에 따라 기능을 구현하는 것이 더 유리
    - 프로젝트 코드가 적을수록 좋다. 만일을 위해 코드를 작성하면 코드베이스의 소유 비용이 불필요하게 증가

### 8.4.2 프로세스 외부 의존성에 인터페이스를 사용하는 이유는 무엇인가?

- 구현체가 하나만 있다고 가정할 때 인터페이스를 사용하는 이유는 목을 사용하기 위함이라는 훨씬 더 실용적이고 현실적인 이유다
- 이는 곧 비관리 의존성에 대해서만 인터페이스를 쓰고(구현체가 단일 일 경우) 관리 의존성을 명시적으로 주입해서 사용함