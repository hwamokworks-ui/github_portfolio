---
description: Notion(Vercel URL 포함)·GitHub 최신 상태로 "Build Log" 포트폴리오 웹페이지를 재생성한다
argument-hint: ""
---

# /build-log

너는 지금부터 아래 단계를 순서대로 수행해 "Build Log" 포트폴리오 웹페이지를 최신 데이터로 다시 만든다. 이 명령은 `/sync-repos`와 별개이며 Notion에 쓰기 작업을 하지 않는다 — Notion·GitHub에서 **읽기만** 하고, 그 결과로 정적 HTML 한 장을 새로 만든다. 이 작업은 CLAUDE.md의 "코드를 작성하지 않는다" 제약과 무관하다 (그 제약은 `/sync-repos`의 Notion 동기화 작업에만 적용된다) — 이미지 압축·폰트 임베드·HTML 조립을 위해 Bash/Python을 자유롭게 사용한다.

Vercel MCP는 호출하지 않는다 — `/sync-repos`가 이미 각 저장소의 `Vercel URL` 속성을 Notion에 채워두므로, 이 페이지는 그 값을 그대로 읽기만 하면 된다.

## 0. 사전 점검
1. Notion MCP로 데이터소스 `collection://e47e4d32-3ee6-49c8-a343-1c7ab4f1159a` ("GitHub 리포지토리 관리")에 접근 가능한지 확인한다.
2. GitHub MCP 연결을 확인한다 (README 조회용).

## 1. 데이터 수집 및 분류
1. **Notion**: `notion-query-data-sources`를 **view 모드**로 호출해 카드 순서의 기준이 되는 뷰를 그대로 조회한다 — `{"mode": "view", "view_url": "https://app.notion.com/p/by1927/fe68680f5a474f6892ef6d39e3663239?v=53bf797693054bc7b6608a4db91bebd3", "page_size": 100}`. 이 뷰는 정렬(sort) 조건이 없는 테이블 뷰라 응답에 담긴 행 순서가 곧 Notion 데이터베이스의 수동 정렬(사용자가 직접 드래그로 배치한) 순서다. **SQL 모드로 별도 조회해 `마지막푸시일` 등으로 다시 정렬하지 않는다** — 이 응답 순서를 그대로 카드 순서로 쓴다. 각 행에서 저장소명, Git URL, 설명, 기술태그, 마지막푸시일, 활동상태, `Vercel URL`, `포트폴리오 노출`을 읽는다. (`has_more`가 true면 `start_cursor`로 이어서 조회해 전체 순서를 놓치지 않는다.)
2. **상태 판정** (Vercel API를 다시 조회하지 않는다):
   - `포트폴리오 노출`이 꺼져 있음(`__NO__`) → **완전히 제외**한다. 포트폴리오 그리드에도, 하단 "미배포" 목록에도 넣지 않는다 — 배포 여부와 무관하게 사용자가 노출을 명시적으로 껐다는 뜻이다. (`/sync-repos`가 새로 추가하는 저장소는 항상 이 상태로 생성되므로, 사용자가 Notion에서 직접 체크하기 전까지는 이 페이지에 자동으로 나타나지 않는다.)
   - `포트폴리오 노출`이 켜져 있고(`__YES__`) `Vercel URL`도 채워져 있음 → **LIVE**. 그 값을 그대로 카드의 Live 링크로 쓴다. (`Vercel URL`은 빌드 실패 상태에서도 채워질 수 있으므로, 이 페이지는 더 이상 BUILD FAIL을 별도로 구분하지 않는다.)
   - `포트폴리오 노출`이 켜져 있고 `Vercel URL`이 비어있음 → **미배포**. 포트폴리오 그리드를 포함해 페이지의 어떤 목록에도 넣지 않는다 (페이지에는 노출하지 않고, 최종 보고 표의 개수 집계에만 반영한다).
3. 카드 순서는 1번에서 받은 view 응답의 행 순서를 그대로 따른다 (임의로 재정렬하지 않는다).

## 2. README 수집 (LIVE 저장소만)
각 저장소에 대해 GitHub MCP `get_file_contents`(owner: 계정명, path: `README.md`)를 호출한다. 404면 그 저장소는 "README 없음"으로 표시한다 (추측해서 내용을 지어내지 않는다). 가져온 원문은 이후 5단계에서 페이지에 JSON으로 그대로 임베드한다 (너무 긴 보일러플레이트성 README는 핵심만 남기고 요약해도 된다).

**카드 제목**: 카드와 모달에 표시되는 제목은 저장소명이 아니라 **README.md의 첫 `#` 제목**을 그대로 쓴다. 단, 그 제목에 **괄호와 그 안의 영문**이 있으면 그 괄호 부분만 제거하고 나머지(한글 등)를 제목으로 쓴다 (예: `AI 공감 다이어리 (AI Empathy Diary)` → `AI 공감 다이어리`). 제목이 온전히 영문이거나 괄호가 없으면 그대로 쓴다. README가 없거나 `#` 제목을 찾을 수 없으면 저장소명을 그대로 쓴다. 두 저장소의 README 제목이 우연히 같아도 임의로 구분하지 않는다.

## 3. 커버 이미지 임베드
Artifact 페이지는 외부 이미지 로드가 CSP로 차단되므로, 반드시 다운로드해서 base64로 직접 파일에 박아 넣어야 한다.
1. 각 LIVE 저장소에 대해 `https://opengraph.githubassets.com/1/{계정}/{저장소명}`을 요청한다. 이 엔드포인트는 분당 요청 제한(약 100/시간, 짧은 시간에 몰리면 429)이 있으니 429가 나면 몇 초 대기 후 재시도한다.
2. Python Pillow로 `Image.open(...).convert("RGB")` → `thumbnail((640, 640))` → `save(..., format="JPEG", quality=68, optimize=True)`로 리사이즈/압축한다 (원본 PNG가 수백 KB~500KB인 경우가 많아 그대로 쓰면 파일이 너무 커진다).
3. base64 인코딩해 `data:image/jpeg;base64,...`로 각 카드의 `<img class="cover">` src에 넣는다. 대량 base64를 Claude 컨텍스트로 읽지 말고, HTML 파일 안의 `__COVER_{TOKEN}__` 같은 플레이스홀더를 Python으로 직접 치환하는 방식을 쓴다 (이전 회차에서 쓴 방식 그대로).

## 4. 폰트 (임베드하지 않는다)
Pretendard 웹폰트 파일을 다운로드·base64 인코딩해 `@font-face`로 박아 넣지 않는다 — 예전 방식은 Regular+Bold 두 weight만으로 파일이 2MB 넘게 불어나 실제 Vercel 배포용 `index.html`이 무거워지는 원인이었다. 대신 `body`를 비롯한 모든 텍스트 요소의 `font-family`를 처음부터 `'Pretendard', -apple-system, BlinkMacSystemFont, "Segoe UI", "Apple SD Gothic Neo", sans-serif`로만 지정한다. `Pretendard`는 어차피 `@font-face` 선언이 없어 항상 폴백되므로 실질적으로는 시스템 폰트로 렌더링되고, 이 방식이 폰트 파일 무게 없이 두 산출물(fragment본·`index.html`) 모두에 동일하게 적용된다.

## 5. 디자인 시스템 (아래 값을 그대로 따른다 — 임의로 다른 색상·폰트로 바꾸지 않는다)

**색상 (라이트 테마 고정, 다크모드 자동 전환 없음)**
```
--paper: #f2f0ea;   --card: #ffffff;
--ink: #1e1c22;     --ink-dim: #625f68;   --ink-faint: #9b98a1;
--line: #dedad2;    --card-line: #c3bcae;  /* 카드 전용 테두리, --line보다 짙음 */
--accent: #4b3f72;
--live: #3f7a5c;  --live-bg: #e6efe8;
--error: #a15c3e; --error-bg: #f5e9e1;
```

**타이포그래피**: 본문 전체는 Pretendard. 코드/README 안의 실제 코드 스니펫(`<code>`, `<pre>`)만 예외로 `ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, monospace` 유지 — 그 외 어디에도 monospace나 serif를 쓰지 않는다 (통계 숫자, 라벨, 칩, 저장소명, 배지, 태그 등 전부 Pretendard).

**아이콘**: 외부 링크 화살표나 닫기 버튼은 유니코드 글리프(↗, ✕) 대신 CSS `mask`(currentColor 기반 SVG data URI) 또는 인라인 `<svg>`를 쓴다 — 폰트에 따라 글자가 깨지는 문제를 원천 차단한다.

**레이아웃 구조**:
- 상단: eyebrow(영문) `2026 Vibe Coding Study` + h1 `Projects` + lede 한 문장만: "카드를 클릭하면 상세 설명을 볼 수 있습니다." (통계 수치나 부연 설명을 lede에 넣지 않는다)
- 탭 2개: **포트폴리오** / **통계**. 상단에 별도 통계 요약 바(전체/라이브/실패/미배포 개수)는 넣지 않는다 — 그런 숫자는 전부 통계 탭 안에만 존재한다.
- 포트폴리오 탭: 기술 스택 필터 칩(라이브 저장소의 기술태그 빈도 기준, 내림차순) → "커버 이미지 보이기" 토글 스위치(**기본값은 꺼짐 = 커버 이미지 숨김**, 켜면 보임) → 카드 그리드. (미배포 저장소 목록은 페이지에 표시하지 않는다 — 위 1-2 참고.)
- 카드 구성: 커버 이미지(2:1 비율, object-fit cover) → 본문(저장소명 + 상태 배지 LIVE, 설명 있으면 표시, 기술 태그, pushed 날짜) → 하단 Live/GitHub 링크 2단 (**GitHub 링크의 라벨은 "GitHub"로 표기한다 — "Code"라고 쓰지 않는다**). **가짜 브라우저 주소창(URL 텍스트 표시 요소)은 넣지 않는다.** 카드 전체가 클릭 가능해야 하고 호버 시 위로 살짝 뜨면서 테두리에 1px box-shadow 링이 추가돼 두꺼워 보여야 한다 (border-width 자체를 바꾸면 레이아웃이 흔들리니 box-shadow로 처리).
- 카드 클릭 → 모달: **커버 이미지는 넣지 않는다** (카드 그리드에만 표시하고 모달 안에는 넣지 않는다). 저장소명, 상태 배지, 설명, 태그, Live/GitHub 링크(**두 버튼은 동일한 스타일**을 쓴다 — 한쪽만 배경을 채운 버튼으로 만들지 않는다. 배경을 채우면 아이콘 currentColor가 배경과 같은 계열이 되어 화살표가 안 보이는 문제가 생겼던 적이 있다), README 렌더링(작은 자체 마크다운→HTML 변환 — 헤딩/굵게/기울임/링크/표/목록/코드펜스 지원, 이미지 마크다운은 원격 이미지를 못 불러오니 `[이미지: alt]` 텍스트로 치환). README 없는 저장소는 "이 저장소에는 README.md가 없습니다."만 보여준다. 모달 안 README 제목(h1~h4)은 반드시 16.5px 이하로 명시적 크기를 줘서 너무 크게 보이지 않게 한다.
- 통계 탭: **`포트폴리오 노출`이 켜진(true) 저장소만 집계 대상으로 한다** (배포 여부는 무관 — 노출 켜짐이면 LIVE든 미배포든 포함, 노출 꺼짐은 완전히 제외). (a) 기술 스택 분포 막대그래프 — 이 집계 대상 저장소들의 기술태그 빈도, 최댓값 기준 막대 폭 비율. (b) 활동상태 비율 — 같은 집계 대상 저장소들의 마지막푸시일 6개월 기준 활발함/방치됨 개수를 스택 바 + 범례로 표시 (0개인 쪽도 숫자로 명시). 이 집계 기준은 화면에 별도 텍스트로 표시하지 않는다 (내부 계산 기준일 뿐).

## 6. 산출물 저장 (fragment본 + 완전한 문서본, 둘 다 필요)
1. **fragment본** (Artifact 발행용): `<title>...</title>`로 시작하고 `<!doctype>`/`<html>`/`<head>`/`<body>` 태그는 절대 넣지 않는다. 스크래치패드에 저장한다.
2. Artifact 도구로 fragment본을 발행한다. 이전에 발행한 "Build Log" 아티팩트가 이 세션에 없다면 `Artifact({action: "list"})`로 기존 것을 찾아 그 URL을 `url` 파라미터로 넘겨서 **같은 링크를 갱신**한다 (URL 없이 발행하면 별개의 새 아티팩트가 생긴다). favicon은 🚀로 고정한다.
3. **완전한 문서본** (실제 배포용 `index.html`): fragment본 내용을 그대로 가져와 아래처럼 감싸서 프로젝트 루트에 `index.html`로 저장한다. **`<meta charset="UTF-8">`이 없으면 실제 배포 시 한글이 깨지므로 반드시 포함한다.**
   ```html
   <!doctype html>
   <html lang="ko">
   <head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1">
   <title>Projects</title>
   </head>
   <body>
   ...fragment본 내용...
   </body>
   </html>
   ```
4. 기존에 프로젝트 루트에 있는 `index_1.html`은 이 작업과 무관한 별개 파일이니 건드리지 않는다.

## 최종 보고
아래를 표로 정리해서 보고한다.

| Notion 전체 | LIVE | 미배포 |
|---|---|---|
| N | N | N |

이어서: Artifact 링크, `index.html` 저장 경로, README를 못 찾은 저장소 목록, 429 등으로 커버 이미지를 못 가져온 저장소가 있으면 그 목록을 덧붙인다.
