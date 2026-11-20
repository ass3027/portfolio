# Hemp 전주기 이력관리 시스템

- 기간: 2024.07 ~ 2024.09
- 역할: BE 개발 지원
- 직급: 연구원

<a id="domain-modeling"></a>

## 건조 프로세스 도메인 로직 설계

건조 상태 전이와 정책 로직이 21개 서비스 클래스에 분산돼 정책을 바꿀 때 확인해야 할 범위가 넓었습니다. 핵심 정책을 상태값 기반의 도메인 모델에 모아 변경 관리 대상을 21개 클래스에서 1개 도메인 모델로 줄였습니다.

<a id="ci-cd"></a>

## GitLab CI/CD 파이프라인 개선

GitLab CI/CD 배포에 15~18분이 걸려 GitLab CI 캐시를 적용하고 개발용과 운영용 Docker 이미지의 중복 빌드를 제거했습니다. GitLab Container Registry를 도입해 내부 이미지와 접근 권한을 관리할 수 있게 했으며, 빌드와 배포 시간을 18분에서 6분으로 줄였습니다.

## 기술 스택

GitLab CI/CD, Spring Boot, JPA, Docker, Docker Swarm
