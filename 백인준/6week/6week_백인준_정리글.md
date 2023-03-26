# [Unit Testing] 5장 목과 테스트 취약성

목을 사용해야하는 의견이 나뉜다. (사용 vs 비사용(리팩터링 내성에 취약))

### 5.1 목 과 스텁의 구분

더미, 스텁 , 스파이, 목 ,페이크 가 있지만 실제로는 두가지 목 과 스텁의 유형

- 5.1.1 테스트 대역 유형
    - 목 (mock)
        - 외부로 나가는 상호작용(SUT가 상태를 변경하기 위한 의존성 호출)을 모방 및 검사에 도움
    - 스텁
        - 내부노 들어오는 상호작용(SUT가 입력 데이터를 얻기 위한 의존성 호출) 모방 하는데 도움
    - 목과 스텁의 차이는 목은 검사까지 하고 스텁은 모방만 한다.
- 5.1.2 도구로서의 목과 테스트 대역으로서의 목

```java
public class example {
    public void Sending_a_greetings_email() {
        var mock = new Mock<>();
        /**
         *
         Mock 클래스는 도구로서의 목이다.
         mock 도구로 생성한 테스트 대역으로서의 목이다.
         도구로서의 목과 테스트 대역으로서의 목 혼동 하지않는다.
         */
        var sut = new Controller();
        sut.GreetUser("email");
        mock.Verify(
                x = > x.SendGreetingsEmail("email"), Times.Once
        );
    }

    public void Creating_a_report() {
        var stub = new Mock<>(); // mock 을 사용하여 스텁 생성
        stub.Setup(x = > x.GetNumberOfUsers())
        .Returns(10);
        //입력 데이터 호출 을 모방한다. stub

        var sut = new Controller(stub.Object);

        Report report = sut.CreateReport();

        Assert.Equal(10, report.NumberOfUsers);
    }
}
```

- 5.1.3 스텁으로 상호 작용을 검증하지 말라
    - SUT에서 스텁으로의 호출은 SUT가 생성하는 최종결과가 아니다. 이러한 호출은 최종결과를 산출하기 위한 수단일뿐
    - 4장에서 말했듯 거짓양성을 피하고 리팩터링 내성을 향상시키는 방법은 테스트 내부 구현사항이 아니라 최종결과를 검증해야 한다.
    - 따라서 최종 결과가 아닌 사항을 검증 하는것을 과잉 명세라하는데 이는 깨지기 쉬운 테스트가 이예이다.
- 5.1.4 목 과 스텁 함께 쓰기
- 하나의 메서드에서 (HasEnoughInventory) 에서 응답을 설정하고 다른메소드의 상호작용을 검증 
```java
public class example2 {
    public void Sending_a_greetings_email() {
        var storeMock = new Mock<>();
        storeMock
                .setUp(x = > x.HasEnoughInventory(product.Shampoo, 5))
        .Returns(false);
        /**
         * 준비된 응답을 설정 stub 인부분
         */
        var sut = new Customer();

        bool success = sut.purchase(storeMock, product.shampoo, 5);

        Assert.False(success);
        storeMock.Verify(
                x = > x.RemoveInventory(product.shampoo, 5), Times.Naver);
        /**
         * sut에서 수행한 호출을 검사
         *
         * HasEnoughInventory 에서 준비한 응답을 가지고 (스텁)
         * removeInventory 를 검사 (목)
         */
    }
}
```
- 5.1.5 목과 스텁은 명령과 조회에 어떻게 관련돼 있는가?
  - 목과 스텁의 개념은 명령 조회 분리 원칙과 관련있다.
  - 목
    - 명령을 대체하는 테스트 대역
    - 사이트 이펙트를 일으키고 어떤값도 반환하지 않는다.(void)
  - 스텁
    - 조회를 대체하는 테스트 대역
    - 사이드 이펙트 없이 값을 반환한다.

###5.2 식별할 수 있는 동작과 구현 세부 사항
- 테스트에 거짓양성이 있는 주요이유는 코드의 구현 세부 사항과 겳합되어있기 떄문이다.
- 테스트는 어떻게가 아니라 무엇에 중점
<br><br>
- 5.2.1 식별할 수 있는 동작은 공개 API와 다르다.
  - 식별할 수 있는 동작
    - 클라이언트가 목표를 달성하는데 도움이 되는 연산을 노출 (계산을 수행 혹은 사이드이펙트를 초래하거나 둘다인 메서드)
    - 목표를 달성하는데 도움이 되는 상태를 노출(상태는 시스템의 현재상태)
  - 모든 구현사항은 클라이언트의 눈에 보이지 않아야한다. (private 메서드)
<br><br>
- 5.2.2 구현 세부 사항 유출 : 연산의 예
  - UserController 에서 이름의 길이가 50 미만인 경우를 체크해서 50자리까지만 보이는 연산이 있는데
  이부분이 public 으로 공개되면 안되고 private 으로 감싸져 연산하는 부분공개 되어야한다.
  - 클라이언트가 목표를 달성하는데 도움이 되는 작업,상태만 노출 하여야한다.
<br><br>
- 5.2.3 잘 설계된 API와 캡슐화 
- 구현 세부 사항을 노출하지 않고 캡슐화 하여야한다.
- 캡슐화
  - 코드베이스 유지 보수에서는 캡슐화가 중요하다.
  - 궁극적으로는 단위테스트와 캡슐화는 동일한 목표를 달성한다.
  - 구현 내부 사항을 숨기면 클라이언트와 시야에서 클래스 내부를 가려 내부를 손상시킬 위험이적다.
  - 데이터와 연산을 결합하면 해당 연산이 클래스의 불변성을 위반하지 않도록 할수 있다.
<br><br>
- 5.2.4 구현세부사항 유출 :상태의 예
  - 식별할수 있는 동작 
    - 공개 : 좋은
    - 비공개 : 해당 없음 
      - 식별할수 있는 동작을 숨기면 클라이언트가 사용할수 없기 때문에 
  - 구현 세부 사항
    - 공개: 나쁨
    - 비공개: 좋음
  - 클라이언트가 목표를 달성하는데 직접적으로 도움이 되는 코드만 공개해야 한다.
