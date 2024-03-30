# 01_비즈니스 도메인 분석하기
## 도메인 주도 설계란

DDD의 전략적, 전술적 패턴과 실무는 모두 소프트웨어 설계를 비즈니스 도메인과 일치시키는 데 사용된다.

→ `(비즈니스) 도메인 주도 (소프트웨어) 설계`

- 비즈니스 도메인을 이해하지 못하면 비즈니스 소프트웨어를 최적화해서 구현하지 못한다.
- 일반적인 프로젝트가 실패하는 이유 → 공통적으로 발견된 주제는 **커뮤니케이션**
    - 불명확한 요구사항, 불명확한 프로젝트 목표, 팀 간의 비효율적인 커뮤니케이션 문제 등

## 비즈니스 도메인

기업의 주요 활동 영역

예시) 스타벅스 : 커피
올리브영 : 뷰티 & 헬스 제품

## 하위 도메인

- 핵심 하위 도메인
    - 흥미로운 문제들. 기업이 경쟁자로부터 차별화하고 경쟁 우위를 얻은 활동
    - 예시 ) 올리브영 - 온오프라인 매장, 다양한 뷰티 브랜드 입점
- 일반 하위 도메인
    - 해결된 문제들. 이것을 모든 회사가 같은 방식으로 하고 있는 일이다. 혁신이 필요하지 않고 사내 솔루션을 개발하는 것보다 기존 솔루션을 사용하는 것이 더 비용 효과적
    - 예시 ) 올리브영 - 암호화, 정산, 인증 및 권한 부여
- 지원 하위 도메인
    - 분명한 해결책이 있는 문제들. 사내에서 구현해야 할 활동이지만 경쟁 우위를 제공하지는 않음.
    - 예시) 올리브영 - 상품/주문 관리, 고객 관리

## 도메인 전문가

우리가 모델링하고 코드로 구현할 비즈니스의 모든 복잡성을 알고 있는 주제 전문가

즉, 소프트웨어의 비즈니스 도메인에 대한 권위자

Product Manager, Business Anlayst, Product Designer 등의 직책을 가진 사람을 말한다. 물론, 개발자 스스로가 기획까지 한다면 개발자 또한 도메인 전문가라고 할 수 있다.