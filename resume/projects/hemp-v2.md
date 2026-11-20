# Hemp 전주기 이력관리 시스템

- 기간: 2024.07 ~ 2024.09
- 역할: BE 개발 지원
- 직급: 연구원

<a id="domain-modeling"></a>

## 건조 상태 전이와 정책 변경 범위 축소

건조 상태 전이와 정책이 21개 서비스 클래스에 흩어져 있어 정책을 바꿀 때 여러 클래스의 동작을 확인해야 했습니다. 건조 상태별 핵심 정책을 도메인 모델에 모아, 정책 변경 시 확인하고 수정할 위치를 한 곳으로 줄였습니다.

<a id="ci-cd"></a>

## GitLab CI/CD 빌드와 배포 시간 단축

GitLab CI에서 의존성 캐시를 적용하고 개발용과 운영용 Docker 이미지의 중복 빌드를 없애 빌드와 배포 시간을 18분에서 6분으로 줄였습니다. GitLab Container Registry를 도입해 내부 이미지의 저장과 접근 권한을 관리했습니다.

## 기술 스택

Spring Boot, JPA, GitLab CI/CD, Docker, Docker Swarm
