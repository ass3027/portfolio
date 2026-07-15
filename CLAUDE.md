# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 저장소의 성격

애플리케이션 코드베이스가 아니라 **개인 개발자 포트폴리오**다. 빌드 시스템·패키지 매니저·테스트 스위트가 없다. 결과물은 브라우저에서 바로 열거나 정적 파일로 서빙하는 **단일 자기완결형(self-contained) HTML 파일**이다.

세 개의 파일이 각각 독립된 결과물이다:

- **`portfolio.html`** — 실제 포트폴리오 사이트. CSS는 하나의 `<style>` 블록에, JS는 하단의 하나의 `<script>` 블록에 인라인된 단일 파일(~1250줄). 한국어 문서(`<html lang="ko">`)이며 이 저장소의 핵심 산출물이다.
- **`PORTFOLIO.md`** — **Tag2Now**(철권 태그 토너먼트 2 실시간 정보 대시보드) 프로젝트의 서술형 케이스 스터디이자 원본 콘텐츠. 여기의 내러티브·아키텍처 다이어그램·코드 발췌·"배운 점"이 `portfolio.html`이 시각적으로 렌더링하는 원천 자료다. 프로젝트 *내용*이 바뀌면 여기를 먼저 수정한다.
- **`랠릿프로필-star+문과 자소.pdf`** — 이력서/자기소개 문서(바이너리, 여기서 편집 불가).

README, `.cursor` 규칙, Copilot 지침은 없다.

## `portfolio.html` 작업 가이드

특정한 내부 구조를 가진 손으로 작성한 단일 파일이다. 수정할 때 이 구조를 유지할 것:

- **섹션**은 `<section id="...">` 블록이다: `about`, `career`, `projects`. 사이드바 `<nav>`의 링크(`href="#id"`)는 이 ID들과 항상 일치해야 한다.
- **스크롤 동작**은 `IntersectionObserver`(1219줄 부근)가 담당한다. 섹션이 뷰에 들어오면 해당 nav 링크에 `.active` 클래스를 토글한다. 섹션을 추가/이름변경하면 `id`와 대응하는 nav `<a href="#...">`를 함께 수정한다.
- **테마**는 `:root`에 CSS 커스텀 프로퍼티(`--bg`, `--accent` 등)로 정의된 GitHub 스타일 다크 팔레트다. 색상은 하드코딩하지 말고 이 변수를 재사용한다.
- **모바일**은 `.mobile-header` + 햄버거 토글(`#hamburger`, 1249줄 부근 핸들러)로 사이드바를 열고 닫는다. 레이아웃을 바꾸면 모바일 폭에서도 동작하는지 확인한다.
- **외부 의존성**은 `<head>`에서 CDN으로 로드된다: Google Fonts(Noto Sans KR, JetBrains Mono)와 highlight.js(`yaml`·`nginx` 언어 팩 포함). 코드 블록은 로드 시 `hljs.highlightAll()`로 하이라이팅된다 — 코드 샘플이 있으면 highlight.js 스크립트와 언어 include를 유지한다.

## 편집 규칙

- 콘텐츠는 **한국어**다. 문구를 추가·수정할 때 기존 어조와 용어를 맞춘다.
- `portfolio.html`은 **자기완결형**으로 유지한다(빌드 단계 없음, CDN 링크 외 로컬 외부 자산 없음). `file://`로 열거나 아무 정적 호스트에 올려도 동작해야 한다.
- `PORTFOLIO.md`의 `[YOUR_ID]`, `[YOUR_DOMAIN]` 플레이스홀더는 의도된 것이다 — 임의의 실제 값으로 채우지 않는다.
- Tag2Now 스토리가 바뀌면 `PORTFOLIO.md`(해당 콘텐츠의 원천)를 먼저 수정하고, 그 뒤 `portfolio.html`에 반영한다.

## 미리보기

`portfolio.html`을 브라우저에서 직접 열거나, 폴더를 정적으로 서빙한다(예: `python -m http.server` 실행 후 해당 파일 접속). 설치나 빌드는 필요 없다.
