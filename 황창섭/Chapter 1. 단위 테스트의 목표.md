﻿# Chapter 1. 단위 테스트의 목표
## 1.1 단위 테스트 현황
많은 개발자들이 단위 테스트의 필요성을 느끼고 변화가 이루어졌지만, 모든 새로운 기술과 마찬가지로 단위 테스트도 발전하며 '좋은 단위 테스트' 라는 모호한 개념이 발생하였다. Unit Test 서적의 목표는 이상적인 단위 테스트 작성을 위해 정확하고 과학적인 정의를 안내한다.

## 1.2 단위 테스트의 목표
테스트 코드의 목표는 결국 지속 가능한 성장이다. 시스템이 발전해나감에 따라 코드 복잡도는 필연적으로 증가하고 결국 침체기에 빠지지만 이를 방지하기 위해선 잘 작성된 테스트 코드가 필요하다. 그리고 테스트 코드도 버그가 발생하고 유지 보수가 필요한 코드이다. 그렇기에 유지 비용을 고려하여 나쁜 테스트를 제거하고 고품질 테스트 코드를 남겨 지속 가능한 프로젝트 성장을 이룰 수 있다.
## 1.3 테스트 스위트 품질 측정을 위한 커버리지 지표
결론은 커버리지 지표는 부정 지표로 이용할 수 있지만 긍정 지표로 이용하기에는 어렵다. 코드 커버리지가 너무 적을 때는 테스트가 충분치 않다는 점을 알 수 있지만, 커버리지가 100%라 해도 양질의 테스트라 기대하기 어렵다. 테스트 스위트의 변경이 없었음에도 코드 리팩토링을 통해 커버리지가 상승할 수 있기 떄문이다. 또 다른 커버리지 지표인 branch coverage는 원시 코드 라인 수를 사용하는 대신 제어 구조에 중점을 둔다. 이는 코드 커버리지보다 더 나은 지표가 될 수 있으나 테스트 스위트의 품질을 결정하는데 의존할 수 없다. 왜냐하면 
- 테스트 대상 시스템의 모든 가능한 결과를 검증한다 보장할 수 없다.
- 외부 라이브러리의 코드 경로를 고려할 수 있는 커버리지 지표는 없다. 

결론적으로 커버리지 숫자를 좋은 테스트 코드를 작성하기 위한 목표로 삼아서는 안된다. 테스트가 부족하다는 것을 나타내기에 커버리지 지표는 아주 좋으나 양질의 테스트가 작성되었음은 전혀 보장하지 못한다.
## 1.4 성공적인 테스트 스위트
- 개발 주기에 통합되어 있다.
- 코드 베이스에서 가장 중요한 부분만을 대상으로 한다.
- 최소한의 유지비로 최대의 가치를 끌어낸다.
### 1.4.1 개발주기에 통합되어 있음
이상적으로 코드가 변경될 때마다 아무리 작은 것이라도 테스트를 해야한다.
### 1.4.2 코드베이스에서 가장 중요한 부분만을 대상으로 함
시스템의 가장 중요한 부분(대체로 비즈니스 로직 쪽)의 단위 테스트 코드 작성에 노력을 기울이고, 다른 부분은 간략하게 또는 간접적으로 검증한다.
### 1.4.3 최소 유지비로 최대 가치
- 가치 있는 테스트를 식별하기
- 가치 있는 테스트 작성하기

비슷한 말처럼 보이지만 가치 높은 테스트를 식별하기 위한 기준틀이 필요하고, 가치 있는 테스트를 작성하기 위해 설계 능력이 필요하다.
## 1.5 Unit Test 책의 목표
- 모든 테스트를 분석하는 데 사용할 수 있는 기준틀을 배운다.
- 테스트를 리팩터링해서 더 가치 있게 만든다.
