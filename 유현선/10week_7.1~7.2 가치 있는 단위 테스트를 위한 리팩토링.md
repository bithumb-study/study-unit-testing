# 7. 가치 있는 단위 테스트를 위한 리팩토링

## 7.1 리팩토링 할 코드 식별하기
테스트 스위트를 크게 개선하는 방법 : 기반코드를 리팩토링 (only)  

### 7.1.1 코드의 4가지 유형
> 리팩토링의 방향을 찾기 위해 코드를 4가지 유형으로 분류

- 제품 코드의 2차원적 분류 
  - 복잡도 또는 도메인 유의성
    - 복잡도 : 코드 내 의사 결정(분기) 지점 수 
    - 도메인 유의성 : 도메인 계층 vs 유틸리티 코드 
    - 복잡한 코드와 도메인 유의성을 갖는 코드가 단위 테스트에서 가장 이로움 
  - 협력자 수 
    - 협력자 : 가변 의존성 or/and 프로세스 외부 의존성


- 도메인 모델과 알고리즘 : **테스트 코드에 가장 이로움**
- 간단한 코드 : 테스트 대상으로 가치가 없음 
- 컨트롤러 : 도메인 클래스와 외부 애플리케이션 같은 다른 구성 요소의 작업을 조종 
- 지나치게 복잡한 코드 : ex) 덩치가 큰 컨트롤러(복잡한 작업을 어디에도 위임하지 않고 모든 것을 스스로하는 컨트롤러)
  - 우리가 리팩토링을 해야 하는 대상 코드   
  → 도메인 모델 및 알고리즘  
  → 컨트롤러 

> 코드가 더 중요해지거나 복잡해질수록 협력자는 더 적어야 한다.


### 7.1.2 험블 객체 패턴을 사용해 지나치게 복잡한 코드 분할하기 

#### 험블 객체 패턴  
- 지나치게 복잡한 코드 → [(테스트 하기 어려운 의존성), (로직)] 클래스를 분리 
- 육각형 아키텍쳐, 함수형 아키텍쳐
- 단일 책임 원칙
- 비즈니스 로직과 오케스트레이션의 분리 

----

## 7.2 가치 있는 단위 테스트를 위한 리팩토링 하기 

### 7.2.1 고객 관리 시스템 (예시 코드)
- 사용자 이메일이 회사 도메인에 속한 경우 해당 사용자는 직원으로 표시 (other : 고객)
- 시스템은 회사의 직원 수를 추적해야 함. 사용자 유형이 변경되면 숫자도 변경
- 이메일이 변경되면 시스템은 메시지 버스로 메시지를 보내 외부 시스템에 알려야 함 

```java
public class User {
	public int userId;
	public String email;
	public UserType type;

	public int getUserId() {
		return userId;
	}
	private void setUserId(int userId) {
		this.userId = userId;
	}

	public String getEmail() {
		return email;
	}

	private void setEmail(String email) {
		this.email = email;
	}

	public UserType getType() {
		return type;
	}

	private void setType(UserType type) {
		this.type = type;
	}


	public void changeEmail(int userId, String newEmail) {
		// 데이터베이스에서 사용자의 현재 이메일과 유형 검색
        // DatabaseConnector 라는 가상의 데이터베이스 객체를가 있다 치고...작성
		User user = DatabaseConnector.getUserById(userId);

		if(user.getEmail().equals(newEmail)) {
			return;
		}

		// 데이터베이스에서 조직 정보(도메인 이름과 직원수) 검색
		Company company = DatabaseConnector.getCompany();
		String emailDomain = newEmail.split("@")[1];
		boolean isEmailCorporate = emailDomain.equals(company.getCompanyDomain());
		UserType newType = isEmailCorporate ? UserType.EMPLOYEE : UserType.CUSTOMER;

		if(user.getType() != newType) {
			// 필요한 경우 조직의 직원 수 업데이트
			int delta = newType == UserType.EMPLOYEE ? 1 : -1;
			int newNumber = company.getNumberOfEmployees + delta;
			company.setNumberOfEmployees(newNumber);
			companyRepository.saveCompany(company);
		}

		// 데이터베이스에 사용자 저장
		user.setEmail(newEmail);
		user.setType(newType);
		userRepository.save(user);

		// 메시지 버스에 알림 전송
		MessageBus.sendEmailChangedMessage(userId, newEmail);
	}

	public enum UserType {
		CUSTOMER,
		EMPLOYEE;
	}
}


```


### 7.2.2 1단계: 암시적 의존성을 명시적으로 만들기 
- 데이터베이스와 메시지 버스에 대한 인터페이스를 두고, 이 인터페이스를 `User`에 주입한 후 테스트에서 목으로 처리
- 도메인 모델은 직접적으로든 간접적으로든 인터페이스를 통해 프로세스 외부 협력자에게 의존하지 않는 것이 훨씬 더 깔끔하다.
- 도메인 모델은 외부 시스템과의 통신을 책임지지 않아야 한다.

### 7.2.3 2단계: 애플리케이션 서비스 계층 도입 


```java
public class UserController {

	private final DatabaseConnector databaseConnector = new DatabaseConnector(); // java 에서의 데이터 연결 구현 생략을 위한 가상의 객체
	private final MessageBus messageBus; // 가상의 객체 22 
	
	public void changeEmail(int userId, String newEmail) {
		User user = databaseConnector.getUserById(userId);
		Company company = databaseConnector.getCompany();
		
		int newNumberOfEmployees = user.changeEmail(newEmail, company.getCompanyDomainName, company.getNumberOfEmployees);
		
		databaseConnector.saveCompany(newNumberOfEmployees);
		databaseConnector.saveUser(user);
		messageBus.sendEmailChangeMessage(userId, newEmail);
	}
}
```

```java
public class User {
	
	// 생략
    
	public int changeEmail(String newEmail, String companyDomainName, int numberOfEmployees) {
		if (email == newEmail) {
			return numberOfEmployees;
		}

		String emailDomain = newEmail.split("@")[1];
		boolean isEmailCorporate = emailDomain.equals(companyDomainName);
		UserType newType = isEmailCorporate ? UserType.EMPLOYEE : UserType.CUSTOMER;

		if (type != newType) {
			int delta = newType == UserType.EMPLOYEE ? 1 : -1 ;
			int newNumber = numberOfEmployees + delta;
			numberOfEmployees = newNumber;
		}
		email = newEmail;
		type = newType;

		return numberOfEmployees;
	}
	
}
```


### 7.2.4 3단계: 애플리케이션 서비스 복잡도 낮추기 
```java
public class UserFactory {
	
	public static User create(Object[] data) {
		assert data.length >= 3;
		
		int id = data[0];
		String email = data[1];
		UserType userType = data[2];
		
		return new User(id, email, userType);
	}
}

```


### 7.2.5 4단계: 새 Company 클래스 소개

도메인 계층의 새로운 클래스 
```java
public class Company {
	public String domainName;
	public int numberOfEmployees;
	
	// getter setter 생략
	
	public void changeNumberOfEmployees(int delta) {
		assert numberOfEmployees + delta >= 0;
		numberOfEmployees += delta;
    }
	
	public boolean isEmailCorporate(String email) {
		String emailDomain = email.split("@")[1];
		return emailDomain == domainName;
    }
}

```


리팩토링 후 컨트롤러
```java 
public class Usercontroller {
	// 가상의 객체...
	private final Database database = new Database();
	private final MessageBus messageBus = new MessageBus(); 
	
	public void changeEmail(int userId, String newEmail) {
		User user = database.getUserById(userId);
		Company company = database.getCompany();
		
		user.changeEmail(newEmail, company);
		
		database.saveUser(user);
		database.saveCompany(company);
		messageBus.sendEmailChangeMessage(userId, newEmail);
    }
}
```


리팩토링 후 User
```java
public class User {
	public int userId;
	public String email;
	public UserType type; 
	
	// getter setter 
	
	public void changeEmail(String newEmail, Company company) {
		if(email.equals(newEmail)) {
			return;
		}

		UserType newType = company.isEmailCorporate(newEmail) ? UserType.EMPLOYEE : UserType.CUSTOMER;
		
		if(type != newType) {
			int delta = newType == USerType.EMPLOYEE ? 1 : -1;
			company.changeNumberOfEmployees(delta);
		}
		email = newEmail;
		type = newType;
	}
}
```