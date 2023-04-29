# [Unit Testing] 7장 가치 있는 단위 테스트를 위한 리팩터링

## 최적의 단위 테스트 커버리지 분석

프로젝트의 모든 코드를 코드 유형 도표의 위치별로 그룹핑
![image](https://user-images.githubusercontent.com/58027908/233811903-3071ba12-b617-4ac8-ac1d-6769529b32e7.png)

### 7.3.1 도메인 계층과 유틸리티 코드 테스트 하기

- 위표의 좌측 상단 테스트 메서드는 비용 편익 측면 최상위 결과 및 코드의 복잡도나 도메인 유의성이 높으면 회귀 방지가뛰어나고 협력자가 거의 없어 유지비도 가장 낮다.

- 전체 커버리지를 달성하려면 세가지 테스트가 더필요
- ```java
    public void Changing_email_from_corporate_to_non_corporate();
    public void Changing_email_without_changing_user_type();
    public void Changing_email_to_the_same_one();    
    ```
- 매개변수화된 테스트로 여러테스트 케이스를 묶을수도 있다. (junit5 parameterizedTest)

```java
public class UserTest {
    @ParameterizedTest
    @MethodSource("testMethods")
    public void Differentiates_a_corporate_meail_form_non_corporate(final String domain, final String email, final boolean expectedResult) {
        var sut = new Company(domain, 0);
        boolean isEmailCorporate = sut.IsemailCorporate(email);
        assertThat(expectedResult, isEmailCorporate);
    }

    private static Stream<Arguments> testMethods() {
        return Stream.of(
                Arguments.of("mycorp.com", "email@mycorp.com", true),
                Arguments.of("mycorp.com", "email@gmail.com", false)
        );
    }
}
```

### 7.3.2 나머지 세 사분면에 대한 코드 테스트 하기

```java
public class User {
    private int userId;
    private String email;
    private UserType type;

    public User(final int userId, String email, UserType type) {
        this.userId = userId;
        this.email = email;
        this.type = type;
    }
}
```

- 생성자는 단순해서 노력을 들일필요가 없어 테스트는 회귀 방지가 떨어질것이다.

### 7.3.3 전체 조건을 테스트 해야 하는가?

```java
    public void ChangeNumberOfEmployee(int delta){
            Preconditions.condition(NumberOfEmployees+delta>=0,"음수X");
            NumberOfEmployees=delta;
        }
```

- 회사 직원수가 음수가 안돼는 전제조건이 있다. (예외 상황에서만 활성화 되는 보호 장치)
- 도메인 유의성이 있는 모든 전체 조건을 테스트하라(직원수가 음수가 되면 안된다는 요구사항은 이러한 전제조건에 해당)
- 도메인 유의성이 없는 전제조건을 테스트하는데 시간을 들이지 말자(전제조건이 없으면 테스트하기에 별 가치가 없다.)

## 7.4 컨트롤러에서 조건부 로직 처리

- 비즈니스 로직과 오케스트레이션 분리는 비즈니스 연산이 세단계로 있을때 가장 효과적
    - 저장소에서 데이터 검색
    - 비즈니스 로직 실행
    - 데이터를 다시 저장소에 저장

![컨트롤러](https://user-images.githubusercontent.com/58027908/233812577-bb77199e-04a3-4ea5-aa22-71f986b610e9.jpg)

- 하지만 단계가 명확하지 않은경우가 있는데 의사 결정 프로세스의 중간 결과를 기반으로 프로세스 외부 의존성에서 추가 데이터를 조회해야 할수도 있다.
- 어쨋든 외부에 대한 모든 읽기와 쓰기를 가장자리로 밀어낸다.
    - 읽고-결정하고-실행하기 구조를 유지하지만 성능이 저하
    - 필요 없는 경우에도 컨트롤러가 프로세스 외부 의존성을 호출한다.
- 도메인 모델에 프로세스 외부 의존성을 주입하고 비즈니스 로직이 해당 의존성을 호출할 시점을 직접 결정할 수 있게 한다.
    - 도메인 모델 테스트 유의성이 떨어진다.
- 의사 결정 프로세스를 좀더 세분화 하고 각 단계별로 컨트롤러를 실행하도록 한다.
    - 컨트롤러가 단순해지지 않는다.

#### [문제는 다음세가지 특성의 균형을 맞춘다.]

- 도메인 모델 테스트 유의성: 도메인 클래스의 협력자 수와 유형에 따른 함수
- 컨트롤러 단순성: 의사 결정 지점이 있는지에 따라 다름
- 성능: 프로세스 외부 의존성에 대한 호출 수로 정의

### 7.4.1 CanExcute/Execute 패턴 사용

- 해당 패턴을 사용해 비즈니스 로직이 도메인 모델에서 컨트롤러로 유출되는 것을 방지하는 것이다.

####[첫번째 도메인 객체안에 결정권]
```java
@Data
public class User {
    private int userId;
    private String email;
    private UserType type;
    private boolean isEmailConfirmed;
    
    public String changeEmail(String newEmail,Company company){
        if(isEmailConfirmed){
            return "Can't change a confirmed email";
        }
        /* 메서드의 나머지 부분 */
    }
}
```
- 해당 구현으로 컨트롤러가 의사결정을 하지않지만 성능저하를 감수해야한다.
- 이메일 변경할수 없는 경우에도 데이터베이스에서 조회를 해야하므로 
- 이는 모든 외부 읽기와 쓰기를 비즈니스 연산끝으로 밀어내는 예
####[두번째 User 에서 컨트롤러로 옮기기]

```java
public class test{
    public String ChangeEmail(int userId,String newEmail){
        User userObject = //data 조회부분;
        if(userObject.isEmailConfirmed){
            return "can not change a confirmed email";
        } // 이부분으로 옮긴다.
    }
}
```
- 성능은 그대로 유지 된다. (이메일이 변경 가능한 후에만 데이터베이스에서 조회)
- 의사결정프로세스가 두부분으로 나뉜다.
  - 이메일 변경 진행 여부
  - 변경시 해야할일
- 모델의 캡슐화가 떨어진다.

####[CanExecute/Execute 패턴 적용]
```java
public class test{
    public String canChangeEmail(){
        if(isEmailConfirmed){
            return "can not change a confirmed email";
        }
        return null;
    }
    
    public void changeEmail(String newEmail, Company company){
      Preconditions.condition(canChangeEmail(), "변경가능");
    }
}
```
- 컨트롤러는 더 이상 이메일 변경 프로세스를 알필요가 없다.
  - 유효성 검사 모두 컨트롤러로부터 캡슐화돼 있다.
- 전제 조건이 추가돼도 먼저 확인하지 않으면 이메일을 변경할수 없다.

### 7.4.2 도메인 이벤트를 사용해 도메인 모델 변경 사항 추적
- 애플리케이션에서 정확히 무슨 일이 일어나는지 외부 시스템에 알려야 하기 때문에 이러한 단계들을 아는 것이 중요할지도 모른다.
  - 컨트롤러에 이러한 책임도 있으면 더 복잡해진다. 이를 피하려면, 도메인 모델에서 중요한 변경 사항을 추적하고 비지니스 연산이 완료된 후 해당 변경 사항을 프로세스 외부 의존성 호출로 변환한다.
  도메인 이벤트(domain event)로 이러한 추적을 구현 할 수 있다.
- 도메인 이벤트는 컨트롤러에서 의사 결정 책임을 제거하고 해당 책임을 도메인 모델에 적용함으로써 외부 시스템과의 통신에 대한 단위 테스트를 간결하게 한다.
- 컨트롤러를 검증하고 프로세스 외부 의존성을 목으로 대체하는 대신, 단위 테스트에서 직접 도메인 이벤트 생성을 테스트할 수 있다.
- 오케스트레이션이 올바르게 수행되는지 확인하고자 컨트롤러를 테스트해야 하지만, 그렇게 하려면 훨씬 더 작은 테스트가 필요하다.

## 7.5 결론
- 외부 시스템에 대한 애플리케이션 사이드 이펙트를 추상화하는 것이었다.
- 비즈니스 연산이 끝날 때까지 이러한 사이드 이펙트를 메모리에 둬서 추상화 프로세스 외부 의존성 없이 단순한 단위 테스트로 테스트 할수 있다.
- 식별할 수 있는 동작이 되려면 메서드는 두가지 기준중 하나 충족 해야한다.
  - 클라이언트 목표 중 하나에 직접적인 연관이 있음
  - 외부 애플리케이션에서 볼 수 있는 프로세스 외부 의존성에서 사이드 이펙트가 발생함

