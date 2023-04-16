# Chapter 7. 가치 있는 단위 테스트를 위한 리팩터링

# Chapter 7. 가치 있는 단위 테스트를 위한 리팩터링

- 1장에서 정의한 좋은 단위 테스트 스위트의 속성
    - 개발 주기에 통합돼 있다
    - 코드베이스 중 가장 중요한 부분만을 대상으로 한다.
    - 최소한의 유지비로 최대의 가치를 끌어낸다
- 7장에서 다루는 내용
    - 네 가지 코드 유형 알아보기
    - 험블 객체 패턴 이해
    - 가치 있는 테스트 작성

## 7.1 리팩터링할 코드 식별하기

- 리팩터링의 방향을 설명하고자 코드를 네 가지 유형으로 분류하는 방법 소개

### 7.1.1 코드의 네 가지 유형

- 모든 제품 코드는 2차원으로 분류
    - 복잡도 또는 도메인 유의성
        - 코드 복잡도: 코드 내 의샤 결정 지점 수로 정의
        - 도메인 유의성: 프로젝트의 문제 도메인에 대해 얼마나 의미 있는지
        - 두 가지의 요소는 독립적이며 복잡하고 도메인 유의성을 갖는 코드일 수록 단위 테스트에 이로움
    - 협력자 수
        - 협력자: 가변 의존성이거나 프로세스 외부 의존성
        - 협력자가 많은 코드는 테스트 비용이 높아짐
- 네 가지 코드 유형
    - 도메인 모델과 알고리즘: 복잡도 및 도메인 유의성이 높고 협력자 수가 적음
        - 단위 테스트가 매우 가치 있고 저렴함
    - 간단한 코드: 복잡도와 도메인 유의성이 낮고 협력자도 거의 없음
        - 테스트의 가치가 0에 가깝기에 단위 테스트가 필요 없음
    - 컨트롤러: 도메인 클래스와 외부 애플리케이션 같은 다른 구성 요소의 작업을 조정. 복잡도는 낮지만 협력자 수가 많음
        - 포괄적인 통합 테스트의 일부로서 간단히 테스트 함
    - 지나치고 복잡한 코드: 복잡도 및 도메인 유의성이 높고 협력자 수도 높은 경우
        - 단위 테스트가 필요하지만 매우 어렵고 위험한 코드

> 좋지 않은 테스트를 작성하는 것보다는 테스트를 전혀 작성하지 않는 편이 낫다.
> 

### 7.1.2 험블 객체 패턴을 사용해 지나치게 복자한 코드 분할하기

- 험블 객체 패턴은 지나치게 복잡한 코드에서 로직을 추출해 코드를 테스트할 필요가 없도록 간단하게 만듬
- 해당 방법은 앞선 장에서 배운 육각형 아키텍처와 함수형 아키텍처에서 이미 사용한 방법

## 7.2 가치 있는 단위 테스트를 위한 리팩터링하기

- 이전 장에서 했던 것과 유사하게 지나치게 복잡한 코드를 알고리즘과 컨트롤러로 나누는 종합 예제를 살펴본다

### 7.2.1 고객 관리 시스템 소개

- 조건
    - 모든 사용자가 데이터베이스에 저장
    - 현재 시스템은 사용자 이메일 변경이라는 단 하나의 유스케이스만 지원
    - 사용자 이메일이 회사 도메인에 속한 경우 해당 사용자는 직원으로 표시된다. 아닐경우 고객으로 간주한다.
    - 시스템은 회사의 직원 수를 추적해야 한다. 사용자 유형이 직원에서 고객으로, 또는 그 반대로 변경되면 이 숫자도 변경해야 한다.
    - 이메일이 변경되면 시슽템은 메시지 버스로 메시지를 보내 외부 시스템에 알려야 한다.
- 초기 구현

```java
public class User {
	private int userId;
	private String email;
	private UserType type;
	
	public void changeEmail(int userId, String newEmail) {
	    // 데이터베이스에서 사용자의 현재 이메일과 유형 검색
	    Object[] data = Database.getUserById(userId);
	    setUserId(userId);
	    setEmail((String)data[1]);
	    setType((UserType)data[2]);
	
	    if (getEmail().equals(newEmail))
	        return;
	
	    // 데이터베이스에서 조직의 도메인 이름과 직원 수 검색
	    Object[] companyData = Database.getCompany();
	    String companyDomainName = (String)companyData[0];
	    int numberOfEmployees = (int)companyData[1];
	
	    String emailDomain = newEmail.split("@")[1];
	    boolean isEmailCorporate = emailDomain.equals(companyDomainName);
	    // 새 이메일의 도메인 이름에 따라 사용자 유형 설정
	    UserType newType = isEmailCorporate ? UserType.Employee : UserType.Customer;
	
	    if (getType() != newType) {
	        int delta = newType == UserType.Employee ? 1 : -1;
	        int newNumber = numberOfEmployees + delta;
	        // 필요한 경우 조직의 직원 수 업데이트
	        Database.saveCompany(newNumber);
	    }
	
	    setEmail(newEmail);
	    setType(newType);
	
	    // 데이터베이스에 사용자 저장
	    Database.saveUser(this);
	    // 메시지 버스에 알림 전송
	    MessageBus.sendEmailChangedMessage(getUserId(), newEmail);
}
```

- 직원과 고객분류, 회사의 직원 수 업데이트에 대한 중요한 명시적 의사결정이 있기에 복잡도와 도메인 유의성 측면에서 점수가 높음
- userId와 newEmail이 명시적 의존성이며 Database와 MessageBus의 암시적 의존성을 포함하기에 협력자 측면에서도 점수가 높다

### 7.2.2 1단계: 암시적 의존성을 명시적으로 만들기

- 암시적 의존성을 명시적으로 변경하는 일반적인 방법은 인터페이스를 두고 User에 주입하는 것이지만 테스트의 관점에서 충분하지 않음
- 결국 도메인 모델은 프로세스 외부 협력자에게 의존하지 않는 것이 좋다

### 7.2.3 2단계; 애플리케이션 서비스 계층 도입

- 도메인 모델이 외부 시스템과 직접 통신하는 문제를 극복하려면 다른 클래스인 험블 컨트롤러로 책임을 옮겨야 함

```java
public class UserController {
    private Database database = new Database();

    private MessageBus messageBus = new MessageBus();

    public void changeEmail(int userId, String newEmail) {
        // 데이터베이스에서 사용자의 현재 이메일과 유형 검색
        Object[] data = Database.getUserById(userId);
        String email = (String) data[1];
        UserType type = (UserType) data[2];
        User user = new User(userId, email, type);

        // 데이터베이스에서 조직의 도메인 이름과 직원 수 검색
        Object[] companyData = database.getCompany();
        String companyDomainName = (String) companyData[0];
        int numberOfEmployees = (int) companyData[1];

        int newNumberOfEmployees = user.ChangeEmail(newEmail, companyDomainName, numberOfEmployees);

        // 데이터베이스에 사용자 저장
        database.SaveCompany(newNumberOfEmployees);
        database.saveUser(user);
        // 메시지 버스에 알림 전송
        message.sendEmailChangedMessage(userId, newEmail);
    }
}
```

- User 클래스는 더 이상 외부 의존성과 통신하지 않기에 테스트가 수월해짐
- 하지만 새로운 UserController 클래스는 컨트롤러 치곤 로직이 여전히 꽤 복잡함

### 7.2.4 3단계: 애플리케이션 서비스 복잡도 낮추기

- UserController가 컨트롤러 사분면에 확실히 있으려면(위에 나온 문제를 해결하려면) 재구성 로직을 추출해야 함.
- ORM 라이브러리를 사용해 데이터베이스를 도메인 모델에 매핑하면, 재구성 로직을 옮기기 적절한 위치가 됨