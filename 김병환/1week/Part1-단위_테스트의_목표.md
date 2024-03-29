# 📚 1장 단위 테스트의 목표

단위 테스트는 단순히 테스트를 작성하는 것보다 더 큰 범주이며, 단위 테스트에 시간을 투자할 때는 항상 최대한 이득을 얻도록 노력해야한다.

- 테스트에 드는 노력 ⬇️ 이득 ⬆️

## 📖 1.1 단위 테스트 현황

___
이제 대부분의 프로그래머는 단위 테스트를 실천하고 중요성을 알고 있다.
예전의 논쟁이었던 '단위 테스트를 작성해야 하는가?'는 '좋은 단위 테스트를 작성하는 것은 어떤 의미인가?'로 바뀌었다.

도움이 될 것이라 생각한 단위테스트가 상황을 악화시키는 경우가 있다. 결국 좋은 단위테스트는 개인의 취향이 아니라 중대한 문제이다.

## 📖 1.2 단위 테스트의 목표

___
단위테스트의 목표: 소프트웨어 프로젝트의 __지속 가능한 성장__ 을 가능하게 하는 것

- 테스트가 없는 프로젝트는 초반에만 발목잡을 것이 없어 빠르게 진척된다. 물론 그 후에는 굉장히 느려진다.
- 소프트웨어 엔트로피(software entropy): 개발 속도가 빠르게 감소하는 현상
- 테스트의 단점은 초반에 상당한 노력이 필요하다는 것

### 🔖 1.2.1 좋은 테스트와 좋지 않은 테스트를 가르는 요인

테스트의 가치와 유지 비용을 모두 고려해야하며 비용 요소는 아래와 같은 활동에 필요한 시간에 따라 결정한다.

- 기반 코드를 refactoring할 때, 테스트도 refactoring하라.
- 각 코드 변경 시 테스트를 실행하라.
- 테스트가 잘못된 경고를 발생시킬 경우 처리하라.
- 기반 코드가 어떻게 동작하는지 이해하려고 할 때는 테스트를 읽는 데 시간을 투자하라.

#### Production Code vs Test Code

- 테스트는 제품 코드에 추가된 것으로 간주되며, 소유 비용이 없다.
- 테스트가 많다고 좋은 것이 아니다.
  - 코드는 자산이 아니라 책임
  - 코드가 많아질수록 유지비 ⬆️, 버그확률 ⬆️
- 테스트코드 또한 유지보수가 필요하다.

## 📖 1.3 테스트 스위트 품질 측정을 위한 커버리지 지표

___

### 커버리지 지표

- 테스트 스위트가 소스 코드를 얼마나 실행하는지를 백분율로 나타낸다.
- 커버리지는 부정 지표로 활용하기는 좋지만 긍정 지표로 활용하기는 힘들다.
  - 100% 달성이라고 좋은 품질의 테스트 스위트라고 보장할 수는 없다.
  - 다만, 적은 달성률은 테스트가 충분하지 않다는 것을 보장한다.

### 🔖 1.3.1 코드 커버리지 지표에 대한 이해
>
> 코드 커버리지(테스트 커버리지) = 실행 코드 라인 수 / 전체 라인 수

- 코드 커버리지는 조작하기가 쉽다.

```java
    public boolean isStringLong(String input) {
        if (input.length() > 5) { // 테스트가 다루는 영역
            return true; // 테스트가 다루지 않는 영역
        }
        return false; // 테스트가 다루는 영역
    }
```

```java
    public boolean isStringLong2(String input) {
        return input.length() > 5;
    }
```

```java
    @Test
    void isStringLongTest() {
        boolean result = example1.isStringLong("abc");
        assertFalse(result);
    }
```

위 예제코드와 같이 같은 결과를 반환하는 코드이지만 코드 커버리지는 다르다.

### 🔖 1.3.2 분기 커버리지 지표에 대한 이해
>
> 분기 커버리지 = 통과 분기 / 전체 분기 수

- 코드가 짧든 짧지 않든 분기 커버리지 지표는 동일하다.
- 코드 커버리지의 단점을 극복하는 데 도움이 된다.

### 🔖 1.3.3 커버리지 지표에 관한 문제점

테스트 스위트의 품질을 결정하는 데 어떤 커버리지 지표도 의존할 수 없다.

- 테스트 대상 시스템의 모든 가능한 결과를 검증한다고 보장할 수 없다.

```java
    private boolean wasLastStringLong;

    public boolean isStringLong3(String input) {
        boolean result = input.length() > 5;
        wasLastStringLong = result; // 첫 번째 결과
        return result; // 두 번째 결과
    }

    @Test
    void isStringLongTest() {
        boolean result = example1.isStringLong("abc");
        assertFalse(result); // 두번째 결과만 검증
    }

    @Test
    @DisplayName("예제 1.3 검증이 없는 테스트는 언제나 통과한다.")
    void isStringLongTest2() {
        boolean result1 = example1.isStringLong("abc");
        boolean result2 = example1.isStringLong("abcdef");
    }
```

- 외부 라이브러리의 코드 경로를 고려할 수 있는 커버리지 지표는 없다.

```java
    /**
     * 외부 라이브러리 경로 고려하지 않는 예제코드
     */
    public int parseExample(String input) {
        return Integer.parseInt(input);
    }

    @Test
    void parseExampleTest() {
        int result = example1.parseExample("5");
        assertEquals(5, result);
    }
```

위와 같은 코드로는 parseInt method가 어떤 validation check를 하는지 확인할 방법이 없다.

### 🔖 1.3.4 특정 커버리지 숫자를 목표로 하기

커버리지 숫자를 강요하면 개발자들은 테스트 대상에 신경 쓰지 못하고, 결국 적절한 단위 테스트는 더욱 달성하기 어려워진다.

- 시스템의 핵심 부분은 커버리지를 높게 두는 것이 좋다. 하지만 이것을 요구사항으로 삼는 것은 좋지 않다.
- 코드 커버리지를 측정하는 것은 좋은 품질로 가기 위한 첫 걸음일 뿐이다.

## 📖 1.4 무엇이 성공적인 테스트 스위트를 만드는가?

___

### 성공적인 테스트 스위트

- 개발 주기에 통합돼 있다.
- 코드베이스에서 가장 중요한 부분만을 대상으로 한다.
- 최소한의 유지비로 최대의 가치를 끌어낸다.

### 🔖 1.4.1 개발 주기에 통합돼 있음

이상적으로는 코드가 변경될 때마다 아무리 작은 것이라도 실행해야 한다.

### 🔖 1.4.2 코드베이스에서 가장 중요한 부분만을 대상으로 함

코드베이스의 모든 부분에 똑같이 주목할 필요는 없다.

- 가장 중요한 부분에 노력을 기울이고, 다른 부분은 간략하게 혹은 간접적으로 검증
- 비즈니스 로직(도메인 모델) 테스트가 최고의 효율이다.

### 🔖 1.4.3 최소 유지비로 최대 가치를 끌어냄

1. 가치 있는 테스트, 가치 낮은 테스트 식별하기
    - 기준틀(frame of reference)이 필요하다.
2. 가치 있는 테스트 작성하기
    - 결국 코드 설계 능력이 필요하다.

## 📖 1.5 이 책을 통해 배우는 것

1. 기준틀(frame of reference)
2. 제품 코드와 관련 테스트 스위트를 리팩터링하는 방법
3. 단위 테스트를 다양한 스타일로 적용하는 방법
4. 통합 테스트로 시스템 전체 동작 검증하기
5. 단위 테스트 안티 패턴을 식별하고 예방하기
