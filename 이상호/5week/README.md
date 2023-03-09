# Unit Testing : 생산성과 품질을 위한 단위 테스트 원칙과 패턴

이 내용은 [단위 테스트 생산성과 품질을 위한 단위 테스트 원칙과 패턴]을 읽으면서 정리한 내용을 포함하고 있습니다.

- 4장 좋은 단위 테스트의 4대 요소 : 4.4 ~ 4.5

목차는 다음과 같습니다.

- 이상적인 테스트를 찾아서
- 대중적인 테스트 자동화 개념 살펴보기
- Action
- 정리

책의 예제는 C#으로 되어 있어서 Java로 변경해서 진행합니다.

## 4장 좋은 단위 테스트의 4대 요소

### 4.4 이상적인 테스트를 찾아서

좋은 단위 테스트의 4대 특성을 다시 살펴보면 다음과 같다.

- 회귀 방지
- 리팩터링 내성
- 빠른 피드백
- 유지 보수성

이 네 가지 특성을 곱하면 테스트의 가치가 결정된다.

- 가치 추정치 = [0..1] * [0..1] * [0..1] * [0..1]
- 가치가 있으려면 테스트는 네 가지 범주 모두에서 점수를 내야 한다.

하지만 이러한 특성을 정확하게 측정하는 것은 불가능하다.

테스트 코드를 포함한 모든 코드는 책임이다.

- 최소한으로 필요한 가치로 임계치를 상당히 높게 설정하고 이 임계치를 충족하는 테스트만 테스트 스위트에 남겨야 한다.
- 소수의 매우 가치 있는 테스트는 다수의 평범한 테스트보다 프로젝트가 계속 성장하는 데 훨씬 더 효과적이다.

#### 4.4.1 이상적인 테스트를 만들 수 있는가?

이상적인 테스트는 네 가지 특성 모두에서 최대 점수를 받는 테스트다.

이상적인 테스트를 만드는 것은 불가능하다.

- 회귀 방지, 리팩터링 내성, 빠른 피드백은 상호 배타적
- 셋 중 하나를 희생해야 나머지 둘을 최대로 가능

특성 중 어느 것도 크게 줄지 않는 방식으로 최대한 크게 해야 한다.

두 특성을 최대로 하는 것을 목표로 해서 한 가지 특성을 희생해 결국 가치가 0에 가까워진 테스트 예시를 살펴보자.

#### 4.4.2 극단적인 사례 1: 엔드 투 엔드 테스트

##### `엔드 투 엔드 테스트` 의 정의

- 최종 사용자의 관점에서 시스템 테스트

##### `엔드 투 엔드 테스트` 특징

- 많은 코드를 테스트하므로 회귀 방지를 훌륭히 해낸다.
  - 프로젝트에서 사용하는 코드를 가장 많이 수행한다.
- 거짓 양성에 면역이 돼 리팩터링 내성도 우수하다.
  - 식별할 수 있는 동작을 변경하지 않는다.
- 어떤 특정 구현도 강요하지 않는다.
  - 최종 사용자의 관점에서 기능이 어떻게 동작하는지만 볼 수 있으며, 구현 세부 사항을 최대한 제거했다.

##### `엔드 투 엔드 테스트` 단점

- 가장 큰 단점은 느린 속도
- 의존하는 모든 시스템은 피드백을 빨리 받기가 어렵다

그래서, 모든 테스트를 `엔드 투 엔드 테스트`만으로 코드베이스를 다루기가 불가능하다.

(그림 133)

- 리팩터링 내성과 회귀 방지에서는 효과가 있음
- 빠른 피드백에서 효과 없음

##### `엔드 투 엔드 테스트` 예시

회원의 엔티티는 다음과 같다.

```java
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
    private String password;
    private Integer age;
    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(
            name = "MEMBER_ROLE",
            joinColumns = @JoinColumn(name = "id", referencedColumnName = "id")
    )
    @Column(name = "role")
    private List<String> roles;
    //..
}
```

회원가입을 하는 API는 다음과 같다.

```java
@RestController
@RequiredArgsConstructor
public class MemberController {
    private final MemberService memberService;

    @PostMapping("/members")
    public ResponseEntity<Void> createMember(@RequestBody MemberRequest request) {
        final MemberResponse member = memberService.createMember(request);
        return ResponseEntity.created(URI.create("/members/" + member.getId())).build();
    }
    //...
}
```

```java
@Service
@RequiredArgsConstructor
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberResponse createMember(final MemberRequest request) {
        Member member = memberRepository.save(request.toMember());
        return MemberResponse.of(member);
    }
    //...
}
```

회원가입의 대한 `엔드 투 엔드 테스트` 시나리오는 다음과 같다.

```text
Feature: 회원 가입

  Scenario: 회원이 신규 가입을 통해 정보를 저장
    Given 회원가입 정보를 입력
    When 회원가입 정보를 요청함
    Then 회원가입 완료를 응답함
```

`엔드 투 엔드 테스트`는 다음과 같이 작성해볼 수 있다.

```java
class MemberAcceptanceTest extends AcceptanceTest {
    @DisplayName("회원가입을 한다.")
    @Test
    void createMember() {
        ExtractableResponse<Response> response = 회원_생성_요청(EMAIL, PASSWORD, AGE);

        회원_생성됨(response);
    }
    //...
}
```

- 인수테스트의 특징 중 하나인 `최종 사용자의 관점에서 기능이 어떻게 동작하는지만 볼 수 있음`이 잘 나타난다.

추가로 다음 항목도 살펴볼 필요가 있다.

```java
public static ExtractableResponse<Response> 회원_생성_요청(String email, String password, Integer age) {
    Map<String, String> params = new HashMap<>();
    params.put("email", email);
    params.put("password", password);
    params.put("age", age + "");

    return RestAssured
            .given().log().all()
            .contentType(MediaType.APPLICATION_JSON_VALUE)
            .body(params)
            .when().post("/members")
            .then().log().all()
            .extract();
    }
```

- `MemberRequest` DTO를 의존하지 않고 Map으로 데이터를 요청하고 있다.
- 이 역시 특정 구현에 의존하지 않고 호출하는 것을 나타낸다.

#### 4.4.3 극단적인 사례 2: 간단한 테스트

다음은 간단한 테스트의 예이다.

```java
@Getter
@Setter
public class User {
    private String name;
}
```

```java
class UserTest {
    @Test
    void testName() {
        var sut = new User();

        sut.setName("John Smith");

        assertThat(sut.getName()).isEqualsTo("John Smith");
    }
}
```

- `엔드 투 엔드 테스트` 와 다르게, 간단한 테스트는 매우 빠르게 실행되고 빠른 피드백을 제공
- 거짓 양성이 생길 가능성이 상당히 낮기 때문에 리팩터링 내성이 우수
- 기반 코드에 실수할 여지가 많지 않기 때문에 간단한 테스트는 회귀를 나타내지 않음

(그림 134)

- 리팩터링 내성과 빠른 피드백에서는 효과가 있음
- 회귀 방지에서 효과 없음

#### 4.4.4 극단적인 사례 3: 깨지기 쉬운 테스트

`깨지기 쉬운 테스트`는 실행이 빠르고 회귀를 잡을 가능성이 높지만 거짓 양성이 많은 테스트를 작성하기가 매우 쉽다.

```java
public class UserRepository {
    public User GetById(int id) {
        /* ... */
    }

    public String lastExecutedSqlStatement;
}

@Test
public void GetById_executes_correct_SQL_code() {
    var sut = new UserRepository();

    User user = sut.GetById(5);

    assertThat("SELECT * FROM dbo.USER WHERE UserId = 5", sut.getLastExecutedSqlStatement());
}
```

- 이 에트스트는 데이터베이스에서 사용자를 가져올 때 UserRepository 클래스가 올바른 SQL 문을 생성하는지 확인
- 이 테스트는 버그를 잡을 수는 있지만 리팩터링 내성은 좋지 못하다.
  - SQL 문을 여러 가지 형태로 변형해도 결과는 모두 같기 때문이다.

(그림 136)

- 빠른 피드백 및 회귀 방지 효과 있음
- 리팩터링 내성 효과 없음

#### 4.4.5 이상적인 테스트를 찾아서: 결론

좋은 단위 테스트의 상호 배타적인 특성

- 회귀 방지
- 리팩터링 내성
- 빠른 피드백

이 세 가지 특성 중 두 가지를 극대화하는 테스트를 만들기는 매우 쉽지만, 나머지 특성 한 가지를 희생해야만 가능하다.

(그림 137)

- 실현 불가능한 테스트

그럼, 어떻게 균형을 맞춰야 할까? 어느 한쪽을 희생해야 할까?

- 회귀 방지, 리팩터링 내성, 빠른 피드백의 상호 배타성 때문에 세 가지 특성 모두를 양보할 만큼 서로 조금씩 인정하는 것이 최선의 전략
- 테스트가 얼마나 버그를 잘 찾아내는지(회귀 방지)와 얼마나 빠른지(빠른 피드백) 사이의 선택으로 귀결된다.
- 리팩터링 내성을 포기할 수 없다. 회귀 방지와 빠른 피드백은 조절이 가능하지만 리팩터링 내성은 조절이 불가능하다.
- `엔드 투 엔드 테스트` 만 쓰거나 테스트가 상당히 빠르지 않은 한, 리팩터링 내성을 최대한 많이 갖는 것을 목표

(그림 138)

- 최상의 테스트는 유지 보수성과 리팩터링 내성을 최대로 갖기 때문에 항상 이 두 특성을 최대화하도록 노력해야 한다.

> 테스트 스위트를 탄탄하게 만들려면 테스트의 불안정성(거짓 양성)을 제거하는 것이 최우선 과제다.

### 4.5 대중적인 테스트 자동화 개념 살펴보기

테스트 피라미드와 화이트박스 테스트 대 블랙박스 테스트에 대해 알아본다.

#### 4.5.1 테스트 피라미드 분해

테스트 피라미드는 테스트 스위트에서 테스트 유형 간의 일정한 비율을 일컫는 개념이다.

(그림 140)

- 넓은수록 해당 테스트는 많아진다.
- 층의 높이는 테스트가 최종 사용자의 동작을 얼마나 유사하게 나타낸다.
  - 사용자의 경험에 가깝게 흉내 내는 것
- 피라미드 내 테스트 유형에 따라 빠른 피드백과 회귀 방지 사이에서 선택을 한다.
- 피라미드 상단의 테스트는 회귀 방지에 유리한 반면, 하단은 실행 속도를 강조한다.

(그림 141)

- 피라미드의 테스트는 빠른 피드백과 회귀 방지 사이에서 선택한다.
- `엔드 투 엔드 테스트`는 회귀 방지에 유리하고, 단위 테스트는 빠른 피드백을 강조하며, 통합 테스트는 그 중간에 있다.

**테스트 유형 간의 정확한 비율**

- 일반적으로 피라미트 형태를 유지
- `엔드 투 엔드 테스트`가 가장 적고, 단위 테스트가 가장 많으며, 통합 테스트는 중간 어딘가에 위치

**`엔드 투 엔드 테스트` 적은 이유**

- 빠른 피드백에 가장 낮은 점수
- 크기가 큰 편이라 프로세스 외부 의존성을 유지하는 데 노력이 필요

`엔드 투 엔드 테스트`는 가장 중요한 기능(버그를 내고 싶지 않은 기능)에 적용할 때와 단위 테스트나 통합 테스트와 동일한 수준으로 보호할 때만 적용된다.

**테스트 피라미드 예외**

- 복잡도가 없는 단순한 CRUD 애플리케이션인 경우
  - 단위 테스트와 통합 테스트는 존재, `엔드 투 엔드 테스트`는 없어도 가능
- 단위 테스트는 알고리즘이나 비즈니스 복잡도가 없는 환경에서는 유용하지 않으므로 간단한 테스트 수준까지 빠르게 내려간다.
- 통합 테스트는 코드가 아무리 단순하더라도 데이터베이스와 같이 다른 하위 시스템과 통합돼 잘 작동하는지 확인하는 것이 중요하다.
- 프로젝트 외부 의존성(ex, 데이터베이스) 하나만 연결하는 API인 경우
  - `엔드 투 엔드 테스트`를 두는 것이 적합한 옵션으로 빠른 피드백이 가능하다.
  - 통합 테스트와 차이없음

**엔드 투 엔드 테스트와 통합 테스트**

- 엔드 투 엔드 테스트는 최종 사용자를 완전히 모방하도록 애플리케이션을 따로 올려서 테스트 진행
- 통합 테스트는 동일한 프로세스에서 테스트 진행

#### 4.5.2 블랙박스 테스트와 화이트박스 테스트 간의 선택

##### 블랙박스 테스트

- 시스템의 내부 구조를 몰라도 시스템의 기능을 검사할 수 있는 소프트웨어 테스트 방법
- 일반적으로 명세와 요구 사항, 즉 애플리케이션이 어떻게 해야 하는지가 아니라 무엇을 해야 하는지를 중심으로 구축

##### 화이트박스 테스트

- 애플리케이션의 내부 작업을 검증하는 테스트 방식
- 요구 사항이나 명세가 아닌 소스 코드에서 파생

##### 각 테스트 장단점

**화이트박스 테스트**

장점으로는 테스트가 더 철저한 편이다.

- 소스 코드를 분석하면 외부 명세에만 의존할 때 놓칠 수 있는 많은 오류를 발견 가능

단점으로는 테스트 대상 코드의 특정 구현과 결합되어서 깨지기 쉽다.

- 거짓 양성을 많이 내고 리팩터링 내성 지표가 부족
- 비즈니스 담당자에게 의미가 있는 동작으로 유추할 수 없다.

블랙박스 테스트는 화이트박스와 정반대이다.

|  |회귀 방지|리팩터링 내성|
|---|---|---|
|화이트박스 테스트|좋음|나쁨|
|블랙박스 테스트|나쁨|좋음|

### 정리

- `엔드 투 엔드 테스트`는 가장 중요한 기능(버그를 내고 싶지 않은 기능)에 적용할 때와 단위 테스트나 통합 테스트와 동일한 수준으로 보호할 때만 적용된다.
- 화이트박스 테스트 대신 블랙박스 테스트를 기본으로 선택하라.
  - 모든 테스트(단위 테스트, 통합 테스트, 엔드 투 엔드 테스트)가 시스템을 블랙박스로 보게 만들고 문제 영역에 의미 있는 동작을 확인하라.
- 테스트를 작성할 때는 블랙박스 테스트가 바람직하지만, 테스트를 분석할 때는 화이트박스 방법을 사용할 수 있다.