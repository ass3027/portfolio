# 덱(Deck) 레이아웃 전환 계획

> **이 문서의 목적**: 새 세션(콜드 스타트)에서 이 작업을 이어받아 실행하기 위한 자기완결형 계획서.
> 브랜치 `deck-layout`에서 진행한다. 먼저 이 문서와 `CLAUDE.md`를 함께 읽고 시작할 것.

---

## 0. 한 줄 요약

`index.html`을 **연속 흐름(continuous-flow) 반응형 웹페이지**에서, **A4 가로 고정 높이 페이지 컨테이너(`.page`)를 쌓은 슬라이드 덱**으로 재구조화한다. 각 `.page` = PDF 한 장. 화면에서 보이는 페이지가 곧 PDF 페이지가 되어 **화면=PDF가 1:1로 완전히 일치**한다.

---

## 1. 왜 하는가 (배경)

- 현재 방식(`main` 브랜치)은 반응형 웹페이지 + 인쇄 시 브라우저 자동 페이지 나눔. 결과 PDF가 **34쪽**이고, 강제 페이지 나눔 + 여백 미압축 때문에 **40~70% 빈 페이지가 다수** 생긴다(1,3,5,11,16,27,30쪽 등). 이건 CLAUDE.md에 "의도된 트레이드오프"로 기록돼 있으나, 제출용 포트폴리오로서는 낭비가 크다.
- 그 외 자동 페이지 나눔의 부작용: 경력 항목이 페이지 경계에서 분리, "KEY CHALLENGES" 같은 section-label이 고아로 남음, LiDAR 프로젝트 카드가 홀로 빈 페이지 차지.
- **덱 방식은 이 문제들을 근본에서 없앤다** — 페이지에 무엇을 넣을지 직접 정하기 때문. 포트폴리오("보여주는 자료")라는 최상위 원칙에도 슬라이드형이 더 잘 맞는다.

---

## 2. 최상위 원칙 (CLAUDE.md 계승 — 반드시 유지)

- **이 프로젝트는 포트폴리오다. 기술 문서가 아니다.** 스캔되는 자료. 성과·수치·아키텍처·핵심 결정을 앞세운다.
- **콘텐츠는 한국어.** 기존 어조·용어 유지.
- **`index.html`은 자기완결형.** 빌드 단계 없음, CDN 링크(Google Fonts, highlight.js) 외 로컬 외부 자산 없음. `file://` 또는 아무 정적 호스트에서 동작.
- **코드 블록(`.code-wrap`)은 숨김**(`display:none`). 코드는 GitHub 몫. **ASCII 아키텍처 박스(`.ascii-box`)는 표시**(추후 SVG/PNG 교체 예정).
- **라이트 테마**(GitHub 라이트 팔레트, `:root` CSS 변수). 색은 하드코딩 금지, 변수 재사용.
- **`PORTFOLIO.md`의 `[YOUR_ID]`·`[YOUR_DOMAIN]` 플레이스홀더는 의도된 것** — 임의 값으로 채우지 않는다. (단, `index.html`에는 이미 실제 값이 들어가 있음: GitHub `ass3027`, LinkedIn 실제 URL, 서비스 `https://match.tag2now.click/`, Tag2Now GitHub `https://github.com/orgs/tag2now/repositories`.)
- **Git: `commit`·`push`는 사용자가 명시적으로 요청했을 때만.** 작업 지시에 커밋을 임의로 덧붙이지 않는다.
- **Hemp 프로젝트는 숨김 상태 유지**(HTML 주석 처리됨). 덱 전환 시에도 미노출로 둔다.

---

## 3. 기술 설계

### 3.1 페이지 컨테이너 CSS

```css
/* @page 여백 0 → .page가 물리 페이지 전체를 채운다 */
@page { size: A4 landscape; margin: 0; }

.page {
  width: 297mm;
  height: 210mm;            /* 정확히 A4 가로 한 장 */
  padding: 12mm 16mm;
  box-sizing: border-box;
  overflow: hidden;         /* 넘치면 잘라냄 (fit은 수동 관리) */
  break-after: page;
  position: relative;
  background: var(--bg);
}
.page:last-child { break-after: auto; }

/* 화면: 회색 배경 위에 페이지를 카드로 미리보기 (PDF 뷰어 느낌) */
@media screen {
  body { background: #333; margin: 0; }
  .page {
    margin: 24px auto;
    box-shadow: 0 4px 24px rgba(0,0,0,.35);
  }
}

@media print {
  body { background: none; }
  .page { margin: 0; box-shadow: none; }
}
```

### 3.2 알려진 함정 (반드시 처리)

1. **서브픽셀 반올림으로 빈 페이지가 생기는 문제**: `.page` 높이가 210mm "정확히"면 렌더링 반올림으로 내용이 1px 넘쳐 빈 페이지가 딸려 나올 수 있다. 대응: 높이를 아주 살짝 줄이거나(예: `209.9mm`), 각 페이지 콘텐츠가 안전 여유를 두게 한다. **PDF로 실제 페이지 수를 세어 검증**(§6).
2. **`overflow:hidden`으로 콘텐츠가 잘림**: 자동 흐름이 아니므로, 한 페이지에 든 내용이 높이를 넘으면 **소리 없이 잘린다.** 문구를 추가/수정할 때마다 fit 확인 필수. (개발 중에는 임시로 `overflow: visible; outline: 1px solid red;`를 켜서 넘침을 눈으로 확인하는 디버그 스위치를 두면 편하다.)
3. **모바일/반응형**: A4 고정 크기는 리플로우되지 않는다. 화면 좁을 때는 `transform: scale()`로 페이지 전체를 뷰포트 폭에 맞춰 축소한다.
   ```css
   @media screen and (max-width: 1200px) {
     .page { transform: scale(calc(100vw / 1130px)); transform-origin: top center; }
     /* scale로 생기는 세로 여백은 margin 음수 보정 또는 wrapper로 처리 */
   }
   ```
   기존 모바일 햄버거/사이드바/스크롤스파이 UX는 덱에서는 제거하거나 "페이지 넘김" 네비게이션으로 대체 검토.
4. **폰트 로딩 타이밍**: 웹폰트(Noto Sans KR)가 늦게 로드되면 fit이 어긋난다. PDF 생성 전 폰트 로드 완료를 보장(headless Chrome은 대체로 처리하나, Playwright 검증 시 `waitForLoadState('networkidle')`).

### 3.3 화면 네비게이션 (결정 필요 — §7)

- 현재: 좌측 사이드바 nav + `IntersectionObserver` 스크롤스파이.
- 덱에서 선택지: (a) 사이드바 유지하되 페이지로 점프, (b) 상/하단 페이지 인디케이터("3 / 20"), (c) 순수 스크롤. **화면=PDF 원칙상 사이드바는 인쇄 시 어차피 숨겨야 하므로, 화면 전용 오버레이 네비로 두는 게 자연스럽다.**

---

## 4. 콘텐츠 → 페이지 매핑 (초안)

현재 `index.html`의 연속 콘텐츠를 아래 페이지들로 재배치한다. **목표: 각 페이지가 꽉 차되 넘치지 않게 균형** — 빈 공간을 없애는 게 이 작업의 핵심 가치.

> **목표: 15쪽 이하(§7-1 확정)**. 아래는 13쪽 압축안. "딥다이브 분량 자체를 줄인다"가 여백 대신 쓰는 정공법. 핵심 프로젝트(Tag2Now·KTX)에 지면을 몰아주고, FireWatcher·LiDAR는 1쪽씩으로 압축.

| # | 페이지 | 담을 내용 |
|---|--------|-----------|
| 1 | **표지 / About** | 프로필 카드(연락처·통계) + "이세진입니다" + bio + 핵심 기술 바 |
| 2 | **경력 & 자격증** | Work Experience(스피어에이엑스) + 자격증 그리드(6개) |
| 3 | **프로젝트 개요** | 4개 카드(Tag2Now/KTX/FireWatcher/LiDAR)를 2×2로 배치 |
| 4 | **Tag2Now — 개요+기술스택** | 헤더 + 프로젝트 배경 + 압축 기술 스택(10 pill) |
| 5 | **Tag2Now — 아키텍처+인프라** | ascii 다이어그램 + 모듈 표 + Before/After + CI/CD를 한 쪽에 압축(넘치면 CI/CD를 덜어냄). **architecture.png 플레이스홀더 제거(§7-4)** |
| 6 | **Tag2Now — 핵심 도전** | 4-1~4-4 도전을 **1쪽에 압축**(problem→result 요약 위주, 코드는 숨김) |
| 7 | **Tag2Now — 배운 점** | learn 카드 6개 |
| 8 | **KTX — 개요+기술스택** | 헤더 + 배경 + 기술 스택(9 pill) |
| 9 | **KTX — 아키텍처 결정 + SLO** | Plan A/B 표 + SLO 표(S1~S6) 압축 |
| 10 | **KTX — 핵심 도전** | 도전 1~4를 **1쪽에 압축** |
| 11 | **KTX — Before/After + 배운 점** | 실험 표(E1~E3) + learn 카드 |
| 12 | **FireWatcher** | 개요+도전+배운점을 **1쪽**으로 압축 |
| 13 | **LiDAR** | 개요+도전+배운점을 **1쪽**으로 압축 |

> **주의**: 위는 추정. 실제로는 각 페이지에 넣어 보고 넘치면(§3.2-2 소리 없이 잘림 주의) 덜어내고, 비면 합친다. 여유 2쪽(→15쪽)은 도전 페이지가 넘칠 때 분할용 버퍼. Hemp는 제외.

---

## 5. 실행 단계 (권장 순서)

> **✅ 2026-07-19 전면 변환 완료.** `index.html`을 13쪽 덱으로 전면 재작성(연속-흐름 34쪽 → 덱 13쪽). PDF 페이지 수 정확히 13(빈 페이지 0), 모든 페이지 넘침/잘림 없음을 §6 방법으로 검증. 사이드바·모바일헤더·스크롤스파이·아코디언·코드블록(`.code-wrap`)·arch-img·highlight.js 제거, 화면 전용 오버레이 인디케이터(◀ n/13 ▶) + 반응형 `zoom` 축소 추가. 남은 일: 사용자 최종 검토, (원하면) 도전 페이지 하단 여백 추가 압축, 커밋. **미커밋 상태.**


1. **프로토타입 먼저** (리스크 조기 검증):
   - `.page` CSS 뼈대 추가.
   - 페이지 1(표지/About) + Tag2Now 개요 1~2쪽만 덱으로 변환.
   - §6 방법으로 화면·PDF 확인. 서브픽셀 빈 페이지·폰트 fit·모바일 스케일 문제를 여기서 잡는다.
2. **네비게이션 결정 및 구현**(§3.3, §7).
3. **나머지 페이지 순차 변환**: About → 경력 → 프로젝트 개요 → Tag2Now → KTX → FireWatcher → LiDAR.
   - 각 페이지 변환 직후 fit 확인(넘침/여백).
4. **기존 연속-흐름 CSS 정리**: 더 이상 안 쓰는 `@media print` 페이지 나눔 규칙, 스크롤스파이 등 제거/대체.
5. **전체 PDF 최종 검증**(§6): 페이지 수, 잘림 없음, 여백 균형.
6. 사용자에게 검토 요청 후, **명시적 지시가 있을 때만** 커밋/푸시.

> 큰 리팩터링이므로 페이지 그룹 단위로 나눠 진행하고 중간중간 확인. 원본은 `main`에 있으니 언제든 참조·롤백 가능.

---

## 6. 검증 방법 (중요 — 이 환경엔 poppler 없음)

PDF 결과는 **추측하지 말고 브라우저로 직접 확인**한다.

1. 로컬 정적 서버: `python -m http.server 8000` (프로젝트 루트에서). `file://`는 Playwright에서 차단됨 → 반드시 http.
2. PDF 생성 (설치된 Chrome 사용):
   ```
   "/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new \
     --no-pdf-header-footer --print-to-pdf="portfolio.pdf" \
     "http://localhost:8000/index.html"
   ```
3. **페이지 수 세기**(빈 페이지 함정 확인):
   ```
   python -c "import re; d=open('portfolio.pdf','rb').read(); print(len(re.findall(rb'/Type\s*/Page[^s]', d)))"
   ```
4. **시각 확인**: 생성한 PDF를 http로 서빙(예: 스크래치패드에서 `python -m http.server 8766`) 후 Playwright로 Chromium 내장 PDF 뷰어에서 열고(`http://localhost:8766/portfolio.pdf`) 스크린샷.
   - 뷰포트를 세로로 크게(예: 1300×2000) 잡고 `page.mouse.wheel(0, 1750)`로 스크롤하며 여러 장 캡처하면 한 번에 여러 페이지를 훑을 수 있다. 좌측 썸네일 레일도 전체 지도로 유용.
   - 임시 스크린샷은 스크래치패드에 저장하고 **프로젝트 폴더를 오염시키지 말 것**(작업 후 `git status`로 확인·정리).
5. 화면(스크린) 자체는 `python -m http.server` + 브라우저로 바로 확인(덱은 화면=PDF라 스크린 확인만으로도 상당 부분 커버됨).

---

## 7. 열린 결정 사항 (2026-07-19 사용자 확정 완료)

1. **분량**: ✅ **15쪽 이하로 공격적 압축**. 딥다이브를 강하게 줄이고 핵심 프로젝트(Tag2Now·KTX) 위주. → §4 매핑을 15쪽 이하로 재조정할 것.
2. **화면 네비게이션**: ✅ **화면 전용 오버레이 인디케이터**("3 / 15" + 상하 점프). 인쇄 시 자동 숨김. 사이드바·스크롤스파이는 제거.
3. **모바일 처리**: `transform: scale`로 페이지 전체를 뷰포트 폭에 맞춰 축소(§3.2-3). 데스크톱 우선.
4. **`architecture.png`**: ✅ **ASCII 박스만 두고 플레이스홀더 제거**. `.arch-img-wrap`/`architecture.png` 관련 마크업·CSS·JS는 삭제하고 `.ascii-box`만 남긴다.
5. **페이지 비율**: **A4 가로 고정**(`@page { size: A4 landscape; margin: 0 }`). 제출이 PDF 인쇄이므로 정석.

### 프로토타입 검증 결과 (2026-07-19, 스크래치패드)
- `.page { height: 209.4mm; padding: 12mm 16mm; overflow: hidden; break-after: page }` + `@page { margin: 0 }` → **정확히 N쪽, 서브픽셀 빈 페이지 0** 확인.
- 가로·세로 잘림 없음. 라이트 팔레트·폰트·컴포넌트 그대로 이식됨. 화면(회색 배경 카드 미리보기)=PDF 1:1 확인.
- 개발용 넘침 디버그 스위치 `body.debug .page { overflow: visible; outline: 1px solid red }` 채택. 페이지 번호 `.page-num`(우하단, 인쇄엔 표시/화면엔 선택).

---

## 8. 참고 (현재 파일 상태)

- **브랜치**: `deck-layout` (이 작업 전용). 원본 연속-흐름 버전은 `main`.
- **`index.html`**: ~1250줄 단일 파일. `:root` 라이트 팔레트 + `--content-max:1000px`. 섹션 `#about`/`#career`/`#projects` + 프로젝트 딥다이브 `#detail-tag2now`/`#detail-ktx`/`#detail-project2`(FireWatcher)/`#detail-lidar`/`#detail-hemp`(숨김).
- **`.pill-*` 색상**: 라이트 테마 대비로 이미 조정됨(진한 색).
- **기술 스택**: Tag2Now는 5그룹→10 pill 한 줄로 압축 완료. KTX/FireWatcher/LiDAR도 한 줄.
- **`PORTFOLIO.md`**: Tag2Now 케이스 스터디 원본. 프로젝트 *내용* 변경은 여기부터.
- **미커밋/미푸시**: `main`의 최근 커밋 `fa09454`(GitHub 표시 텍스트 수정)까지가 로컬 상태. 필요 시 `main` 참조.

---

*이 계획은 시작점이다. 실제 콘텐츠를 페이지에 넣어보며 매핑·분량을 조정한다. 애매하면 항상 "포트폴리오로서 이게 맞나?"로 되돌아간다.*
