# PRD: Vercel 배포 저장소 카드형 포트폴리오 — "Build Log" (`/build-log`)

- 작성일: 2026-08-05 (최종 갱신: 2026-08-15 — 데이터 출처와 산출물 형태가 초기 설계에서 크게 바뀌어 전면 갱신됨)
- 실행 방식: `/build-log` 슬래시 명령. 코드를 작성하지 않는다는 제약과 무관하며(그 제약은 `/sync-repos`의 Notion 동기화 작업에만 적용), 이미지 압축·폰트 처리·HTML 조립에 Bash/Python을 자유롭게 사용한다.
- 산출물 형태: **두 가지 모두 생성한다** — ① Artifact로 발행하는 공유용 fragment 페이지, ② 실제 Vercel 배포용 완전한 문서 `index.html` (프로젝트 루트에 저장)
- 데이터 출처: **Notion "GitHub 리포지토리 관리" DB가 주 출처다.** `/sync-repos`가 이미 채워둔 `Vercel URL`·`포트폴리오 노출`·기술태그·마지막푸시일 등을 그대로 읽는다. GitHub MCP는 README 원문과 커버이미지 원본(OG 이미지) 요청에만 보조적으로 쓴다. **Vercel MCP는 호출하지 않는다** — Notion에 이미 검증된 `Vercel URL`을 재사용하므로 다시 조회할 필요가 없다.

> **변경 이력**: 최초 설계(2026-08-05)는 "GitHub MCP만 사용, Notion과 완전히 무관, `homepage` 필드로 Vercel 배포 판정, 저장소 내 `cover.*` 파일 우선 탐색"이었다. 하지만 `/sync-repos`가 이미 Notion에 더 정확한 `Vercel URL`(Vercel API 직접 조회 기반)과 `포트폴리오 노출` 플래그를 관리하게 되면서, 같은 판정을 GitHub 메타데이터만으로 다시 추론하는 것은 이중 작업이자 오차 소지였다. 그래서 데이터 출처를 Notion으로 전환했다. 아래 내용은 이 전환 이후의 최신 동작 기준이다.

---

## 1. 배경 및 문제

GitHub 계정에 흩어진 저장소 중 실제로 **Vercel에 배포되어 남에게 링크로 보여줄 수 있는 저장소**가 무엇인지는 저장소를 하나하나 열어보지 않으면 알 수 없다. `/sync-repos`가 이 정보를 Notion DB에 이미 정리해두고 있으므로, 이를 재료로 커버이미지·기술스택·README 요약·실제 배포 링크가 담긴 카드형 웹페이지를 만들어 외부에 공유할 "포트폴리오 한 장"을 완성한다.

## 2. 목표

1. Notion DB에서 `포트폴리오 노출`이 켜져 있고 `Vercel URL`이 채워진 저장소 **전부**를 골라, 커버이미지·이름·설명·기술스택 배지·실제 배포(Vercel) 링크·GitHub 링크가 담긴 카드 그리드로 보여주는 페이지를 만든다.
2. 카드를 클릭하면 커버 이미지 확대 + README 렌더링을 볼 수 있는 모달을 제공한다.
3. 카드 순서는 임의 정렬이 아니라 **지정된 Notion 뷰의 수동(드래그) 순서**를 그대로 따른다.
4. 링크 하나로 공유 가능한 Artifact와, 실제로 Vercel에 배포 가능한 가벼운 정적 `index.html` 두 가지를 모두 만든다.
5. 라이트 테마로 고정한다(다크모드 자동 전환은 이번 범위에 없다).

### 성공 지표 (Definition of Done)
- [ ] `포트폴리오 노출=true` + `Vercel URL` 있음 조건을 만족하는 저장소를 **빠짐없이** 카드로 표시한다.
- [ ] `포트폴리오 노출=false`인 저장소는 배포 여부와 무관하게 완전히 제외된다(그리드에도, 미배포 목록에도 없음).
- [ ] 각 카드에 커버이미지, 저장소명, 설명(있는 경우), 기술 태그, LIVE 상태 배지, Live/Code 링크, pushed 날짜를 보여준다.
- [ ] 카드 순서가 지정된 Notion 뷰(`https://app.notion.com/p/by1927/fe68680f5a474f6892ef6d39e3663239?v=53bf797693054bc7b6608a4db91bebd3`)의 행 순서와 정확히 일치한다.
- [ ] `Vercel URL`은 있지만 `포트폴리오 노출`이 꺼진 저장소, `포트폴리오 노출`은 켜졌지만 `Vercel URL`이 없는 저장소가 각각 올바른 위치(완전 제외 / "미배포" 목록)에 반영된다.
- [ ] Artifact로 발행되어 공유 가능한 URL이 생성되고, 재실행 시 같은 URL이 갱신된다.
- [ ] 프로젝트 루트의 `index.html`이 base64 폰트·이미지 없이 가벼운 상태로(외부 이미지 URL 참조, 시스템 폰트 폴백)로 저장된다.
- [ ] 제작 과정에서 Vercel MCP 도구를 전혀 호출하지 않는다.

## 3. 범위 (Scope)

### 3.1 In Scope
| 영역 | 내용 |
|---|---|
| 데이터 조회 | Notion `notion-query-data-sources`를 **view 모드**로, 카드 순서 기준 뷰(정렬 없는 테이블 뷰)를 조회한다. 이 응답이 곧 전체 데이터 + 카드 순서 + 통계 탭 집계 대상을 동시에 제공한다 |
| 필터링 | `포트폴리오 노출=false` → 완전 제외. `포트폴리오 노출=true` + `Vercel URL` 있음 → LIVE 카드. `포트폴리오 노출=true` + `Vercel URL` 없음 → "미배포" 목록 |
| 정렬 | 위 조회 응답의 행 순서를 그대로 사용(재정렬하지 않음) — 사용자가 Notion에서 직접 드래그로 배치한 순서 |
| README | LIVE 저장소만 GitHub MCP `get_file_contents`로 조회, 404면 "README 없음" |
| 커버이미지 | GitHub OG 이미지(`https://opengraph.githubassets.com/1/{owner}/{repo}`)만 사용(저장소 내 `cover.*` 파일 우선 탐색은 더 이상 하지 않음). fragment본은 base64 임베드, `index.html`은 외부 URL 그대로 참조 |
| 폰트 | Pretendard 웹폰트를 다운로드·base64 임베드하지 않는다. 시스템 폰트 폴백 스택(`'Pretendard', -apple-system, BlinkMacSystemFont, "Segoe UI", "Apple SD Gothic Neo", sans-serif`)만 지정 |
| 산출물 | ① Artifact fragment(공유용) ② `<!doctype>`부터 감싼 완전한 문서 `index.html`(Vercel 배포용, 프로젝트 루트 저장) — 매 실행 둘 다 생성/갱신 |
| 데이터 갱신 | 수동 — `/sync-repos`로 Notion이 갱신되거나 카드 순서 뷰가 바뀌면 `/build-log`를 다시 실행해 두 산출물을 모두 재발행/재저장한다 |

### 3.2 Out of Scope
- Notion 쓰기 작업 일체 (조회만 함 — `/sync-repos`와 역할 분리)
- Vercel MCP 직접 호출을 통한 실시간 배포 상태 재확인 (Notion의 `Vercel URL`을 신뢰) — 하위 호환을 위해 `/sync-repos`가 최신 상태를 유지해야 함
- 저장소 내 `cover.*` 파일 탐색 (과거엔 우선 사용했으나 폐기 — 항상 GitHub OG 이미지만 사용)
- GitHub 변경을 감지해 자동으로 재생성하는 파이프라인
- 다크 테마 자동 전환
- 커스텀 도메인 연결 여부 확인 (Notion의 `Vercel URL` 값을 그대로 신뢰)
- 모바일 앱(Expo) 등 Vercel과 무관한 배포 형태 포함

## 4. 카드 순서 기준 (Notion 뷰)

카드 순서는 정렬(sort) 조건이 없는 특정 테이블 뷰의 행 순서를 그대로 따른다:
`https://app.notion.com/p/by1927/fe68680f5a474f6892ef6d39e3663239?v=53bf797693054bc7b6608a4db91bebd3`

이 뷰에 정렬 조건이 없기 때문에 응답 순서가 곧 Notion 데이터베이스에서 사용자가 직접 드래그로 배치한 수동 순서다. `notion-query-data-sources`를 `mode: "view"`로 호출해 이 뷰의 `view_url`을 그대로 전달하면 이 순서를 얻을 수 있다. **SQL 모드로 별도 조회해 `마지막푸시일` 등 다른 기준으로 재정렬하지 않는다.**

## 5. 상세 요구사항

### FR-1. 데이터 조회 (Notion view 모드, 단일 조회로 순서·필터·통계 동시 확보)
- `notion-query-data-sources`를 `{"mode": "view", "view_url": "<4장의 URL>", "page_size": 100}`로 호출한다. `has_more`가 true면 `start_cursor`로 이어서 조회해 전체 순서를 놓치지 않는다.
- 각 행에서 저장소명, `Git URL`, 설명, 기술태그, 마지막푸시일, 활동상태, `Vercel URL`, `포트폴리오 노출`을 읽는다.
- 이 응답은 필터가 없는 전체 데이터셋이므로, 통계 탭(FR-6의 (a)(b))에 필요한 "Notion 전체 저장소 기준" 집계에도 그대로 재사용한다 — 별도 SQL 조회를 하지 않는다.

### FR-2. 상태 판정 (Vercel API를 다시 조회하지 않음)
- `포트폴리오 노출`이 꺼짐(`__NO__`) → **완전히 제외**. 그리드에도, 하단 "미배포" 목록에도 넣지 않는다.
- `포트폴리오 노출`이 켜짐(`__YES__`) + `Vercel URL` 있음 → **LIVE**. `Vercel URL`을 그대로 카드의 Live 링크로 쓴다(빌드 실패 상태에서도 채워질 수 있어 BUILD FAIL을 별도로 구분하지 않는다).
- `포트폴리오 노출`이 켜짐 + `Vercel URL` 없음 → **미배포**. 그리드에는 넣지 않고 페이지 하단 "Notion에는 있지만 아직 배포되지 않음" 목록에 GitHub 링크만 표시한다.
- 카드 순서, "미배포" 목록 순서 모두 FR-1의 조회 응답 순서를 그대로 따른다.

### FR-3. README 수집 (LIVE 저장소만)
- GitHub MCP `get_file_contents`(path: `README.md`)로 원문을 가져온다. 404면 "README 없음"으로 표시(추측 금지). 너무 긴 보일러플레이트성 README는 핵심만 남기고 요약해도 된다.

### FR-4. 커버 이미지
- 각 LIVE 저장소의 `https://opengraph.githubassets.com/1/{owner}/{repo}`를 사용한다(429 발생 시 몇 초 대기 후 재시도).
- **fragment본(Artifact)**: Artifact CSP가 외부 이미지 로드를 막으므로, Pillow로 `640×640` 이내 리사이즈 + JPEG quality 68 압축 후 base64로 임베드한다. 대량 base64를 Claude 컨텍스트로 직접 읽지 않고, 플레이스홀더 토큰을 Python으로 치환하는 방식을 쓴다.
- **`index.html`(실배포용)**: CSP 제약이 없으므로 base64 임베드 없이 OG 이미지 URL을 그대로 `<img src>`에 참조한다 — 이 차이가 파일 크기를 최소 수백 KB~2MB 이상 줄인다.

### FR-5. 폰트 (임베드하지 않음)
- Pretendard 웹폰트 파일을 다운로드·base64 인코딩해 `@font-face`로 넣지 않는다(과거 방식은 Regular+Bold 두 weight만으로 2MB 이상 불어나 실배포용 `index.html`을 무겁게 만드는 주원인이었다).
- 모든 텍스트 요소의 `font-family`를 처음부터 `'Pretendard', -apple-system, BlinkMacSystemFont, "Segoe UI", "Apple SD Gothic Neo", sans-serif`로 지정한다 — `@font-face` 선언이 없어 항상 시스템 폰트로 자연스럽게 폴백된다.

### FR-6. 페이지 구성
- 상단: eyebrow `2026 Vibe Coding Study` + h1 `Projects` + lede 한 문장("카드를 클릭하면 상세 설명을 볼 수 있습니다.")
- 탭 2개: **포트폴리오** / **통계**
  - 포트폴리오 탭: 기술 스택 필터 칩(빈도 내림차순) → "커버 이미지 보이기" 토글(기본 꺼짐) → 카드 그리드 → "미배포" 목록
  - 카드: 커버이미지(2:1) → 저장소명 + LIVE 배지 + 설명(있으면) + 기술 태그 + pushed 날짜 → Live/Code 링크. 카드 전체 클릭 가능, 클릭 시 모달(커버·저장소명·상태·설명·태그·Live/GitHub 링크·README 마크다운 렌더링)
  - 통계 탭: (a) 기술 스택 분포 막대그래프 — **Notion 전체 저장소 기준**(배포 여부 무관) (b) 활동상태 비율(활발함/방치됨) 스택 바 + 범례
- 라이트 테마 색상 고정: `--paper:#f2f0ea` `--ink:#1e1c22` `--accent:#4b3f72` `--live:#3f7a5c` `--error:#a15c3e` 등 (다크모드 전환 없음)

### FR-7. 산출물 저장 (fragment본 + 완전한 문서본, 둘 다 필요)
- **fragment본**: `<title>`로 시작, `<!doctype>`/`<html>`/`<head>`/`<body>` 태그 없이 스크래치패드에 저장 후 Artifact 도구로 발행(기존 발행 URL이 있으면 그 URL로 갱신, favicon 🚀 고정).
- **`index.html`(실배포용)**: fragment본을 `<!doctype html><html lang="ko"><head><meta charset="UTF-8">...</head><body>...</body></html>`로 감싸 프로젝트 루트에 저장한다. `<meta charset="UTF-8">` 누락 시 배포 환경에서 한글이 깨지므로 반드시 포함한다.
- 실제 Vercel 배포(GitHub 연동 프로젝트에 push하거나 `deploy_to_vercel`로 새 프로젝트 생성)는 `/build-log` 자체 범위가 아니라 별도 요청 시 수행한다 — `/build-log`는 배포 가능한 상태의 `index.html`을 만드는 것까지가 책임이다.

## 6. 리스크

| 항목 | 내용 |
|---|---|
| Notion 의존성 | `/sync-repos`가 먼저 실행되어 `Vercel URL`·`포트폴리오 노출`·기술태그가 정확히 채워져 있어야 한다. `/sync-repos`를 실행하지 않았거나 오래돼 정보가 낡았으면 이 페이지도 낡은 정보를 보여준다 |
| 신규 저장소는 기본적으로 미노출 | `/sync-repos`가 새로 발견한 저장소는 항상 `포트폴리오 노출=false`로 생성된다(기존 노출 여부·카드 순서를 자동으로 되돌리지 않기 위함). 사용자가 Notion에서 직접 체크하기 전까지는 `/build-log`를 재실행해도 이 페이지에 나타나지 않는다 — "빠짐없이 표시"는 노출이 켜진 저장소에 한정된 보장이다 |
| 카드 순서 기준 뷰 삭제/변경 위험 | 4장의 특정 뷰 URL이 삭제되거나 필터가 추가되면 카드 순서 조회가 실패하거나 의도와 다른 부분집합만 반환될 수 있다 |
| 정적 스냅샷 | Artifact와 `index.html` 모두 실행 시점 데이터로 고정된다. Notion이 바뀌면(노출 토글, 신규 배포 등) `/build-log`를 다시 실행해 두 산출물을 모두 재생성해야 최신화된다 |
| 커버이미지 | 항상 GitHub OG 이미지(리포지토리 메타데이터 기반)로, 실제 배포 화면 스크린샷이 아니다 |
| README 렌더링 한계 | 자체 경량 마크다운→HTML 변환기를 쓰므로 복잡한 GFM 문법(중첩 목록, 각주 등)은 일부 깨질 수 있다. 이미지 마크다운은 렌더링 대신 `[이미지: alt]` 텍스트로 대체된다 |
| 429 (OG 이미지) | 짧은 시간에 몰리면 요청이 실패할 수 있어 재시도 로직이 필요하다 |
| fragment본과 index.html의 이미지 처리 차이 | fragment본은 base64 임베드, index.html은 외부 URL 참조로 서로 다르다 — 한쪽만 수정하고 다른 쪽을 안 맞추면 두 산출물이 어긋난다 |

## 7. 향후 확장 아이디어 (이번 범위 밖)
- `/build-log`가 만든 `index.html`을 실제로 Vercel에 자동 배포까지 하는 단계 통합 (현재는 파일 생성까지만, 배포는 별도 요청)
- Vercel Web Analytics 등으로 실제 방문 데이터를 페이지에 노출
- GitHub/Notion 변경 감지 → `/build-log` 자동 재실행 파이프라인
- Vercel 외 배포처(Netlify, GitHub Pages 등)로 카드 그룹 확장
- 다크 테마 지원
