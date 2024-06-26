# 04_바운디드 컨텍스트 연동

바운디드 컨텍스트는 서로 독립적이지 않고 상호작용해야 한다.

## 협력형 패턴 그룹
협력형 그룹의 패턴은 소통이 잘 되는 팀에서 구현된 바운디드 컨텍스트와 관련이 있다.

### 파트너십

바운디드 컨텍스트는 **애드혹 방식으로 연동**된다.

연동의 조정은 양방향에서 한다. 발생할 수 있는 연동의 문제를 해결하는 데 양 팀 모두 협력한다.

### 공유 커널

두 개 이상의 바운디드 컨텍스트에서 참여하는 모든 바운디드 컨텍스트가 공유하는 **겹치는 모델을 공유해서 연동**한다.

- 언제 사용할까?
    - 지리적 제약이나 조직의 정치적 문제로 커뮤니케이션 또는 협업이 어려워서 **파트너십 패턴을 구현하기 어려울 때** 구현한다.
    - **레거시 시스템**을 점진적으로 현대화할 경우다. 이런 상황에서는 시스템을 서서히 바운디드 컨텍스트로 분해해서 공유 코드베이스로 만드는 것이 실용적인 중간 솔루션이 될 수 있다,

## 사용자-제공자 패턴 그룹
제공자는 사용자에게 서비스를 제공한다. 서비스 제공자는 ‘업스트림(upstream)’이고 고객 또는 사용자는 ‘다운스트림(downstream)’이다.

### 순응주의자

사용자는 서비스 제공자의 모델에 순응한다.

### 충돌 방지 계층(ACL : anticorruption layer)

사용자는 서비스 제공자의 모델을 사용자의 요건에 맞게 번역한다.

### 오픈 호스트 서비스(OHS : open-host service)

충돌 방지 계층 패턴의 반대로 서비스 제공자는 사용자의 요건에 최적화된 모델인 공표된 언어를 구현된다.

## 분리형 노선
협력과 연동보다 특정 기능을 중복으로 두는 것이 더 저렴한 경우다. 하지만 핵심 하위 도메인을 연동할 경우에는 협업 없는 분리형 노선은 피해야 한다.