# PRD: Vercel 배포 저장소 카드형 포트폴리오 (독립 웹페이지)

- 작성일: 2026-08-05
- 산출물 형태: **독립 웹페이지(Artifact, HTML/CSS)**
- 데이터 출처: **GitHub MCP로 직접 조회한 데이터만 사용한다.** Notion과는 완전히 무관하며, 기존 `/sync-repos`(GitHub↔Notion 동기화 워크플로우)와도 별개로 동작한다 — Notion DB를 읽거나 쓰지 않는다.

---

## 1. 배경 및 문제

GitHub 계정에 흩어진 저장소 중 실제로 **Vercel에 배포되어 남에게 링크로 보여줄 수 있는 저장소**가 무엇인지는 저장소를 하나하나 열어보지 않으면 알 수 없다. 이걸 외부에 공유할 "포트폴리오 한 장"으로 만들려면, 커버이미지와 실제 배포 주소가 함께 들어간 카드 UI로 Vercel 배포가 확인된 **전부**를 보여주는 독립 페이지가 필요하다.

## 2. 목표

1. GitHub 계정의 저장소 중 "Vercel에 배포된" 저장소 **전부**를 골라, 커버이미지·이름·설명·기술스택·**실제 배포(Vercel) 주소**가 담긴 카드 그리드로 보여주는 웹페이지(Artifact)를 만든다.
2. 링크 하나로 공유 가능해야 한다 (Artifact 발행 URL).
3. 라이트/다크 테마 모두에서 깨지지 않아야 한다.
4. 모든 데이터는 GitHub MCP 조회만으로 완결한다 (Notion 등 다른 시스템에 대한 의존이 없다).

### 성공 지표 (Definition of Done)
- [ ] Vercel 배포가 확인된 저장소를 **빠짐없이** 카드로 표시한다 (일부만 골라 담지 않는다).
- [ ] 각 카드에 커버이미지, 저장소명, 설명(있는 경우), 기술스택 배지, **GitHub 코드 링크와 실제 Vercel 배포 링크를 모두** 보여준다.
- [ ] 배포 확인된 저장소가 0개여도 페이지가 깨지지 않고 안내 문구를 보여준다.
- [ ] Artifact로 발행되어 공유 가능한 URL이 생성된다.
- [ ] 제작 과정에서 Notion MCP 도구를 전혀 호출하지 않는다.

## 3. 범위 (Scope)

### 3.1 In Scope
| 영역 | 내용 |
|---|---|
| 대상 저장소 | GitHub 계정 저장소 중 "Vercel 배포 근거"가 있는 것 **전체** (4장 기준, 이번 확인 시점 기준 17개) |
| 카드 구성 | 커버이미지(우선순위: ① 저장소 안에 `cover`로 시작하는 이미지 파일이 있으면 그것 ② 없으면 `opengraph.githubassets.com` OG 이미지), 저장소명, 설명(있으면), 기술스택 배지, **실제 Vercel 배포 주소 링크**, GitHub 코드 링크 |
| 정렬 | 마지막 푸시일(`pushed_at`) 내림차순 |
| 산출물 | Artifact 도구로 발행한 정적 HTML 1페이지 |
| 데이터 갱신 | 수동 — GitHub 저장소 내용이 바뀌면(새로 Vercel 배포, cover 이미지 추가 등) 이 프롬프트를 다시 요청해 Artifact를 재발행 |

### 3.2 Out of Scope (이번엔 제외)
- Notion 연동 일체 (조회·기록 모두 하지 않음)
- Vercel API/토큰 연동을 통한 실시간 배포 상태(빌드 성공 여부, 최근 배포 시각 등) 확인 — 권한 없음
- GitHub 변경을 감지해 자동으로 재생성하는 파이프라인
- 실제 배포 화면 스크린샷 캡처 (커버이미지는 저장소 내 `cover` 파일 또는 GitHub OG 이미지로 대체)
- 커스텀 도메인 연결 여부 확인 (`*.vercel.app` 기본 주소만 신뢰, 커스텀 도메인으로 갈아탄 경우는 놓칠 수 있음)
- 모바일 앱(Expo) 등 Vercel과 무관한 배포 형태 포함

## 4. "Vercel에 배포됨" 판정 기준과 대상 (17개)

**판정 근거 (변경됨)**: 처음에는 저장소 루트의 `vercel.json` 존재 또는 `package.json`의 `vercel` 의존성으로 판정했다. 그런데 이 방식으로는 **book-movie-tracker, fridgechef, onkki 3개만 걸러졌고 나머지 14개를 놓쳤다** — Vercel이 GitHub 앱 연동으로 배포되면 저장소에 설정 파일을 전혀 남기지 않기 때문이다. 그래서 판정 기준을 GitHub 저장소 메타데이터의 **`homepage` 필드가 `*.vercel.app` 도메인을 가리키는지**로 바꿨다. 이 필드는 Vercel이 GitHub 저장소와 연결될 때 자동으로 채워지므로, 설정 파일 유무와 무관하게 실제 연결 여부를 훨씬 정확히 반영한다.

전체 21개 저장소(계정 소유, `english-life-quotes-app`은 사용자가 삭제해 제외) 중 `homepage`가 `*.vercel.app`인 17개:

| 저장소 | Vercel 주소 | 비고 |
|---|---|---|
| visitiedcafemap | visitiedcafemap.vercel.app | 저장소 자체 `cover.png` 있음 |
| dessert_cafe | dessert-cafe-smoky.vercel.app | 저장소 자체 `cover.png` 있음 |
| simple_alarm | simple-alarm-taupe.vercel.app | 저장소 자체 `cover.png` 있음 |
| english-life-quotes-web | english-life-quotes-web.vercel.app | private 저장소 |
| book-movie-tracker | book-movie-tracker.vercel.app | |
| ai-empathy-diary | ai-empathy-diary-amber.vercel.app | |
| onkki | onkki.vercel.app | |
| fridgechef | fridgechef-ten.vercel.app | |
| visitiedcafemap-pwa | visitiedcafemap-pwa.vercel.app | |
| incheon-airport-congestion-dashboard | incheon-airport-congestion-dashboar.vercel.app | private 저장소 |
| selfcare_v4 | selfcare-v4.vercel.app | |
| selfdailycare | selfdailycare.vercel.app | |
| healthcheck | healthcheck-gamma.vercel.app | |
| selfcare | selfcare-opal.vercel.app | |
| moana | moana-peach.vercel.app | 설명 근거 없어 카드에 캡션 생략 |
| portfolio | portfolio-two-sand-w01002vkix.vercel.app | |
| cafe | cafe-seven-mu.vercel.app | 20개 분석 대상 밖이었던 저장소지만 Vercel 배포가 확인돼 포함 |

`homepage`가 비어 있어 **제외**한 4개: vibe-hiking-reservation, jejucafemap, jongno-elibrary-dashboard, vibecoding (Python 스크립트형이거나 파일 몇 개짜리 빈 저장소로, 배포 자체가 없음).

## 5. 상세 요구사항

### FR-1. 데이터 조회 (GitHub MCP만 사용)
- GitHub MCP로 계정의 전체 저장소 목록을 조회한다 (페이지네이션 끝까지, `pushed_at` 내림차순 정렬).
- 각 저장소의 `homepage` 필드를 확인해 `*.vercel.app`으로 끝나면 대상에 포함한다 (4장 기준).
- 대상 저장소마다 저장소명·GitHub URL·description·pushed_at·주요 언어·`homepage`(Vercel 주소)를 가져온다.
- 사설(private) 저장소도 판정 대상에 포함한다 — 다만 코드 링크는 로그인 없이는 열리지 않을 수 있음을 카드에 표시한다 (FR-4).

### FR-2. 커버이미지 결정 (우선순위 규칙)
- 대상 저장소마다 GitHub MCP로 루트 및 하위 폴더(2단계 깊이)를 조회해, 파일명이 `cover`로 시작하는 이미지 파일(`cover.png`, `cover.jpg`, `cover.jpeg`, `cover.webp`, `cover.svg` 등, 대소문자 무관)이 있는지 찾는다.
- **있으면**: 해당 파일의 GitHub raw URL(`https://raw.githubusercontent.com/{owner}/{repo}/{branch}/{path}`)을 카드 커버이미지로 사용한다.
- **없으면**: GitHub OG 이미지 URL(`https://opengraph.githubassets.com/1/{owner}/{repo}`)을 그대로 구성해 사용한다.
- 2단계 깊이보다 더 깊은 곳에 있는 `cover` 파일은 이번 범위에서 찾지 않는다 (탐색 비용 제한).
- `search_code`(코드 검색) 결과는 방금 push된 파일을 바로 반영하지 못할 수 있으므로, 최근에 push된 저장소는 `get_file_contents`로 루트 디렉터리를 직접 조회해 재확인한다.

### FR-3. 기술스택 배지
- 저장소의 `package.json` 의존성, 파일 확장자, README 언급 내용 등 GitHub에서 직접 확인 가능한 근거로만 기술스택 배지 목록을 구성한다 (근거 없는 항목은 추측해서 채우지 않는다).
- "Vercel"은 이 페이지에 실리는 모든 저장소가 공통으로 만족하는 조건이라 배지로 반복 표시하지 않는다 (구분에 도움이 안 되는 정보는 생략).

### FR-4. 카드 레이아웃
- 카드 하나 = 커버이미지(상단, FR-2 규칙으로 결정) + 저장소명(제목, private면 옆에 "private repo" 표시) + 설명(있을 때만) + 기술스택 배지 목록 + **실제 Vercel 배포 주소 링크(주 링크)** + GitHub 코드 링크(보조 링크) + 마지막 배포일.
- description이 없는 저장소는 설명 영역 자체를 생략한다 (빈 문구를 노출하지 않는다).
- private 저장소는 카드에 "private repo" 배지를 붙이고, 코드 링크가 로그인 없이는 열리지 않을 수 있음을 페이지 하단 설명에 명시한다.

### FR-5. 그리드/반응형
- 데스크톱 3열, 태블릿 2열, 모바일 1열 그리드.
- 라이트/다크 테마 모두 대응 (`artifact-design` 스킬 가이드 준수).

### FR-6. 빈 상태
- Vercel 배포 확인 저장소가 0개면 "아직 Vercel에 배포된 저장소가 없습니다" 안내만 보여주고 에러 없이 렌더링한다.

### FR-7. 발행
- Artifact 도구로 발행하고 공유 가능한 URL을 사용자에게 전달한다.
- 파비콘은 프로젝트 성격에 맞는 이모지 1~2개로 지정한다.

## 6. 리스크

| 항목 | 내용 |
|---|---|
| 판정 정확도 | `homepage` 필드가 `*.vercel.app`인지만 확인한다 — 실제로 그 URL이 지금도 살아있는지(빌드 실패, 프로젝트 삭제 등)는 요청해보지 않는다 |
| 커스텀 도메인 누락 | Vercel 프로젝트에 커스텀 도메인을 연결하면서 `homepage`를 그 도메인으로 바꿔둔 경우, `*.vercel.app` 판정에서 빠질 수 있다 |
| 정적 스냅샷 | Artifact는 발행 시점 데이터로 고정된다. GitHub 저장소가 바뀌면(신규 Vercel 배포, cover 이미지 추가 등) 이 프롬프트를 다시 실행해 같은 파일 경로로 재발행해야 최신화된다 |
| 커버이미지 | `cover` 파일이 없으면 실제 배포 화면이 아니라 GitHub OG 이미지(리포지토리 메타데이터 기반)로 대체된다 |
| cover 파일 탐색 범위 | 2단계 깊이까지만 확인하므로, 더 깊은 폴더(예: `src/assets/cover.png`)에 있으면 못 찾고 OG 이미지로 폴백할 수 있다 |
| 코드 검색 인덱스 지연 | 당일 새로 push된 파일은 `search_code`에 안 잡힐 수 있어, 최근 push 저장소는 반드시 `get_file_contents`로 직접 재확인해야 한다 (FR-2) |
| private 저장소 접근성 | private로 표시된 2개는 GitHub 코드 링크를 다른 사람이 열면 로그인/권한 오류가 날 수 있다 — Vercel 배포 링크는 공개이므로 그대로 열린다 |
| Notion과의 데이터 불일치 가능성 | 이 페이지는 Notion "GitHub 리포지토리 관리" DB를 참조하지 않고, 그 DB가 다루는 20개 범위와도 다르다(cafe 포함, 일부 미분석 저장소 제외) — 서로 다른 결과물로 의도된 것이다 |

## 7. 향후 확장 아이디어 (이번 범위 밖)
- Vercel API/토큰 연동으로 실제 배포 상태·최근 배포 시각을 자동 확인
- 커스텀 도메인까지 포함하도록 판정 기준 보강 (Vercel API로 프로젝트-도메인 매핑 조회)
- GitHub 저장소 변경 감지 → Artifact 자동 재발행 파이프라인
- Vercel 외 배포처(Netlify, GitHub Pages 등)로 카드 그룹 확장
