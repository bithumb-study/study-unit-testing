# 5장 목과 테스트 취약성

mock 은 훌륭한 도구이며 대부분의 테스트에 적용해야 한다  
vs  
mock 이 테스트 취약성을 초래하며 사용하지 말아야한다

> 2장 내용 복습  
> 고전파 vs 런던파
> - 고전파 : 상향식 TDD
>   - 단위 테스트를 분리해서 병렬로 실행해야 한다
>   - 격리 주체 : 단위 테스트 - 테스트를 어떤 순서(병렬이나 순차 등)로든 가정 적합한 방식으로 실행할 수 있으며 서로의 결과에 영향을 미치지 않는다.
> - 런던파 : 하향식 TDD
>   - 목 추종자(mockist)
>   - 테스트 대역 사용 대상 : 불변 의존성 외 모든 의존성
>   - 동작을 외부 영향과 분리해서 테스트 대상 클래스에만 집중


## 5.1 mock 과 stub 구분 

### 5.1.1 테스트 대역 유형
- 테스트 대역 : 모든 유형의 비운영용 가짜 의존성
  - Mock : Mock, Spy
  - Stub : Stub, Dummy, Fake


[ mock 과 stub 의 차이점 ]
- Mock : 외부로 나가는 상호 작용을 모방하고 검사 - SUT 가 상태를 변경하기 위한 의존성을 호출하는 것에 해당
  - ex) SUT 에서 이메일을 발송(SMTP 서버 호출) 하는 부분을 mocking -> 부작용을 초래하는 외부 상호작용 
  - SUT 와 관련된 의존성 간의 상호작용을 모방하고 검사 
- Stub : 내부로 들어오는 상호 작용을 모방 - SUT 가 입력 데이터를 얻기 위한 의존성을 호출하는 것에 해당 
  - ex) SUT 에서 데이터베이스의 데이터를 검색하는 부분을 Stub -> 부작용을 일으키지 않음
  - 모방만 함 
- [마틴 파울러 - Mock 은 Stub 이 아니다 ](https://martinfowler.com/articles/mocksArentStubs.html)

[ 그 외 ]
- mock - spy : spy 는 수동으로 작성된 mock / mock 은 프레임워크(JUnit - mockito)
- stub - dummy - fake : 얼마나 똑똑한지 
  - dummy : 단순하고 하드코딩된 값
  - stub : 정교함. 시나리오마다 다른 값을 반환하게 구성된 필요한 것을 다 갖춘 완전한 의존성
  - fake : 대다수 목적에 부합하는 stub. 차이점은 보통 아직 존재하지 않은 의존성을 대체하고자 구현 


### 5.1.2 도구로서의 mock 과 테스트 대역으로서의 mock
mock 프레임워크를 사용해서 생성했다고 해서 모두 같은 mock 이 아님, 사용에 따라 Stub 이 될 수 있음
```java
public class MockArentStubTest {

    // mock - 외부로 나가는 상호 작용을 모방
    // ex) SUT 에서 이메일을 발송(SMTP 서버 호출) 하는 부분을 mocking -> 부작용을 일으킴

    @InjectMocks
    private SutController sutController;
    
    // @Mock
    // private IEmailGateway iEmailGateway;

    @Test
    void sendingAGreetingsEmail() {
        // given - mock 
        IEmailGateway iEmailGateway = mock(IEmailGateway.class);
        
        // when 
        sutController.greetUser("user@email.com");
        
        // then: 테스트 대역으로 하는 SUT 의 호출 검사 
        verify(iEmailGateway.sendGreetingsEmail("user@email.com"));
    }


    // stub - 내부로 들어오는 상호 작용을 모방
    // ex) SUT 에서 데이터베이스의 데이터를 검색하는 부분을 Stub -> 부작용을 일으키지 않음
    @Test 
    void creatingReport() {
        // stub
        UsersRepository usersRepository = mock(UsersRepository.class);
        
        // 준비한 응답 설정 : given
        when(usersRepository.getNumberOfUsers()).thenReturn(10);
        
        // when 
        Report report = sutController.createReport();
        
        // then 
        assertEquals(10, report.numberOfUsers());
    }
}
```

### 5.1.3 Stub 으로 상호 작용을 검증하지 말라
- Stub 은 SUT 가 출력을 생성하도록 입력을 제공
- 테스트에서 거짓 양성을 피하고 리팩토링 내성을 향상시키기 위해서는 구현 세부사항이 아니라 최종 결과를 검증해야 함 
- 호출을 검증하는 것은 테스트 취약성으로 이어짐 (깨지기 쉬운 테스트)
  - `verify(iEmailGateway.sendGreetingsEmail("user@email.com"));`
- 과잉 명세 : 최종 결과가 아닌 사항을 검증하는 관행

```java
// 과잉명세 예시 
public class BadCase_Overspecification {
    // 위 코드 sub 과 일부 동일 
    @Test
    void creatingReport() {
        // stub
        UsersRepository usersRepository = mock(UsersRepository.class);
        when(usersRepository.getNumberOfUsers()).thenReturn(10);

        Report report = sutController.createReport();

        // then -> 과잉명세! 
        verify(usersRepository.getNumberOfUsers());
    }
}
```

### 5.1.4 Mock 과 Stub 함께 쓰기
```java
public class MockAndStub {
    
    @Test
    void purchaseFailsWhenNotEnoughInventory() {
        IStore storeMock = mock(IStore.class);
        when(storeMock.hasEnoughInventory(Product.Shampoo, 5)).thenReturn(false); // 준비된 응답 설정
        Customer sut = new Customer();
        
        boolean success = sut.purchase(storeMock.hasEnoughInventory(Product.Shampoo, 5));
        assertFalse(success);
        verify(storeMock.removeInventory(Product.Shampoo, 5)).naver();
    }
}
```

### 5.1.5 Mock 과 Stub 은 명령과 조회에 어떻게 관련돼 있는가? 
- 명령 조회 분리 (CQS, Command Query Separation) 원칙
  - 모든 메서드는 명령이거나 조회여야 하며, 이 둘은 혼용해서는 안됨 
  - 명령 : 부작용 초래 | 반환 값 없음 -> Mock
  - 조회 : 부작용 없음 | 값 반환 -> Stub


---

## 5.2 식별할 수 있는 동작과 구현 세부 사항 
[ 앞 내용 요약 ]
- 단위 테스트 시 리팩토링 내성을 최대한 활용해야 함 
- 코드가 생성하는 최종 결과(식별할 수 있는 동작)를 검증하고 구현 세부 사항과 테스트를 가능한 한 떨어뜨려야 함
- 테스트는 '어떻게'가 아니라 '무엇'에 중점을 둬야 함 

### 5.2.1 식별할 수 있는 동작은 공개 API 와 다르다
- 공개 API or 비공개 API 
- 식별할 수 있는 동작 or 구현 세부 사항 


- 식별할 수 있는 동작
  - 클라이언트가 목표를 달성하는 데 도움이 되는 **[ 연산 or 상태 ]** 를 노출
    - 연산 : 계산을 수행하거나 부작용을 초래하거나 둘 다 하는 메서드
    - 상태 : 시스템의 현재 상태


- 잘 설계된 API 
  - 공개 API = 식별할 수 있는 동작
  - 구현 세부 사항 = 비공개 API


### 5.2.2 구현 세부 사항 유출 : 연산의 예 
구현 세부 사항을 노출하는 User Class
```java
/**
 * - 사용자 이름이 50자를 초과하면 X 
 * - 초과하면 잘라야 한다(불변속성)
 */
@Setter
@Getter
public class User {
    
    private String name;
    
    public String normalizeName(String name) {
        String result = StringUtils.trum(name);
        if(result.length() > 50) {
            return result.substring(0, 50);
        }
        return result;
    }
}

public class UserController {
    
    public void RenameUser(int userId, String newName) {
        User user = getUserFromDatabase(userId);
        
        String normalizeName = user.normalizeName(newName);
        user.getName() = normalizeName;
        
        saveUserToDatabase(user);
    }
}

```
식별할 수 있는 동작이 아닌 `normalizeName` 메서드를 노출하고 있음  


API 가 잘 설계된 User 클래스
```java
public class User {
    
    private String name;
    
    public String getName() {
        return normalizeName(name);
    }
    
    public void setName(String name) {
        this.name = normalizeName(name);
    }
    
    public String normalizeName(String name) {
        String result = StringUtils.trum(name);
        if(result.length() > 50) {
            return result.substring(0, 50);
        }
        return result;
    }
}

public class UserController {
    
    public void RenameUser(int userId, String newName) {
        User user = getUserFromDatabase(userId);
        user.setName(newName);
        saveUserToDatabase(user);
    }
}

```


### 5.2.3 잘 설계된 API 와 캡슐화
- 캡슐화 : 불변성 위반 - 모순을 방지하는 조치 
- 장기적으로 코드베이스 유지 보수에서는 캡슐화가 중요 : 계속 증가하는 코드 복잡도에 대처
- 구현 세부 사항을 숨기면 클라이언트의 시야에서 클래스 내부를 가릴 수 있기 때문에 내부를 손상시킬 위험이 적다
- 데이터와 연산을 결합하면 해당 연산이 클래스의 불변성을 위반하지 않도록 할 수 있다


### 5.2.4 구현 세부 사항 유출: 상태의 예
- 좋은 단위 테스트와 잘 설계된 API 사이에는 본질적인 관계가 있다
- 모둔 구현 세부 사항을 비공개로 하면 테스트가 식별할 수 있는 동작을 검증하는 것 외에는 다른 선택지가 없으며, 리팩토링 내성도 자동으로 좋아진다.
- 연산과 상태를 최소한으로 노출
- 클라이언트가 목표를 달성하는 데 직접적으로 도움이 되는 코드만 공개해야 하며, 다른 모든 것은 구현 세부 사항이므로 비공개 API 뒤에 숨겨야 함 

