# Chapter 9. 목 처리에 대한 모범 사례

**9장에서 다루는 내용**

- 목의 가치를 극대화하기
- 목을 스파이로 교체하기
- 목 처리에 대한 모범 사례

목의 사용

- 목은 SUT와 의존성 간의 상호 작용을 모방하고 검사하는 데 도움이 되는 테스트 대역
- 목은 비관리 의존성에만 적용해야 함

## 9.1 목의 가치를 극대화하기

```java
public class EventDispatcher {
    private MessageBus messageBus;
    private DomainLogger domainLogger;
    
    public void dispatch(List<DomainEven> events){
        for (DomainEvent ev:
             events) {
            Dispatch(ev);
        }
    }
    
    private void dispatch(DomainEvent ev){
        switch (ev){
            case EmailChangedEvent emailChangedEvent:
                messageBus.sendEmailChangeMessage(emailChangedEvent.getUserId(), emailChangedEvent.getNewEmail());
                break;
            case UserTypeChangedEvent userTypeChangedEvent:
                domainLogger.userTypeHasChanged(
                        userTypeChangedEvent.getUserId(),
                        userTypeChangedEvent.getOldType(),
                        userTypeChangedEvent.getNewType()
                );
                break;
        }
    }
}
```

```java
public class IntegrateTest{

    public void changing_email_from_corporate_to_non_corporate(){
        // 준비
        Database db = new Database();
        User user = User.createUser();
        Company.createCompany();

        Mock messageBusMock = new Mock<MessageBus>();
        Mock loggerMock = new Mock<DomainLogger>();

        UserController sut = new UserController(db, messageBusMock, loggerMock);

        // 실행
        String result = sut.changeEmail();

        // 검증
        Assert.Equal("OK", result);

        ...

        messageBusMock.verify(
                x -> x.sendEmailChangedMessage(user.getUserId(), "new@gmail.com"), Times.Once
        );
        loggerMock.verify();
    }
}
```

### 9.1.1 시스템 끝에서 상호 작용 검증하기

- 위의 통합 테스트에서 사용했던 목이 회귀 방지와 리팩터링 내성 측면에서 이상적이지 않음
- messageBusMock의 문제점은 MessageBus 인터페이스가 시스템 끝에 있지 않다는 것
- MessageBus는 Bus 인터페이스의 Wrapper 클래스
- MessageBus에서 Bus로 대상을 바꾼 후의 통합 테스트는 아래와 같다

```java
public class Test{

    public void changing_email_from_corporate_to_non_corporate(){
        // 준비
        Database db = new Database();
        User user = User.createUser();
        Company.createCompany();

        Mock busMock = new Mock<Bus>();
        MessageBus messageBus = new MessageBus(busMock.class);
        Mock loggerMock = new Mock<DomainLogger>();

        UserController sut = new UserController(db, messageBus, loggerMock);

        // 실행
        String result = sut.changeEmail();

        // 검증
        Assert.Equal("OK", result);

        ...

        busMock.verify(
                x -> x.send("Type: USER EMAIL CHANGED; " + $"Id: {user.getUserId()}; " + "NewEmail: new@gmail.com"), Times.Once
        );
        loggerMock.verify();
    }
}
```

- 외부 시스템은 애플리케이션으로부터 텍스트 메시지를 수신하고, MessageBus와 같은 클래스를 호출하지 않음
- 텍스트가 외부에서 식별할 수 있는 유일한 사이드 이펙트
- 이런식으로 구현하면 리팩터링 내성이 향상되고(MessageBus는 얼마든지 수정할 수 있으므로) 회귀 방지가 좋아짐

### 9.1.2 목을 스파이로 대체하기

- 스파이는 직접 작성한 목과 같음
- 시스템 끝에 있는 클래스의 경우 스파이가 목보다 낫다
- 스파이는 검증 단계에서 코드를 재사용해 테스크 코드를 줄이고 가독성을 향상

```java
public class Test{

    public void changing_email_from_corporate_to_non_corporate(){
        // 준비
        Database db = new Database();
        User user = User.createUser();
        Company.createCompany();

        BusSpy busSpy = new BusSpy();
        MessageBus messageBus = new MessageBus(busSpy);
        Mock loggerMock = new Mock<DomainLogger>();

        ...

        busSpy.shouldSendNumberOfMessages(1)
              .withEmailChangedMessage(user.UserId, "new@gmail.com");
    }
}
```

- BusSpy와 목을 사용한 경우 비슷해 보이지만 BusSpy는 테스트 코드에 MessageBus는 제품 코드에 속한다.
- 테스트에서 검증문을 작성할 때 제품 코드에 의존하면 안 되므로 이 차이는 중요

## 9.2 목 처리에 대한 모범 사례

- 목 처리의 모범사례
    - 비관리 의존성에만 목 적용하기
    - 시스템 끝에 있는 의존성에 대해 상호 작용 검증하기
    - 통합 테스트에서만 목을 사용하고 단위 테스트에서는 하지 않기
    - 항상 목 호출 수 확인하기
    - 보유 타입만 목으로 처리하기

### 9.2.1 목은 통합 테스트만을 위한 것

- 목은 비관리 의존성에만 해당하며 컨트롤러만 이러한 의존성을 처리하는 코드기에 통합 테스트에서 컨트롤러를 테스트할 때만 목을 적용한다

### 9.2.2 테스트장 목이 하나일 필요는 없음

- 앞에서도 나왔듯 단위는 동작의 단위이기에 단일 클래스부터 여러 클래스에 이르기까지 다양하게 걸쳐 있을 수 있다
- 목 또한 마찬가지로 여러개가 존재할 수 있다

### 9.2.3 호출 횟수 검증하기

- 비관리 의존성과의 통신에 관해서는 예상하는 호출이 있는지와 예상치 못한 호출이 없는지 두 가지 모두 확인해야 함

### 9.2.4 보유 타입만 목으로 처리하기

- 서드파티 라이브러리 위에 항상 어댑터를 작성하고 기본 타입 대신 해당 어댑터를 목으로 처리해야 한다
- 이유
    - 서드파티 코드의 작동 방식에 대해 깊이 이해하지 못하는 경우가 많다.
    - 해당 코드가 이미 내장 인터페이스를 제공하더라도 목으로 처리한 동작이 실제로 외부 라이브러리와 일치하는지 확인해야 하므로, 해당 인터페이스를 목으로 처리하는 것은 위험하다.
    - 서드파티 코드의 기술 세부 사항까지는 꼭 필요하지 않기에 어댑터는 이를 추상화하고, 애플리케이션 관점에서 라이브러리와의 관계를 정의한다.