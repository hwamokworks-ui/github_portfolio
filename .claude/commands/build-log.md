---
description: Notion(Vercel URL 포함)·GitHub 최신 상태로 "Build Log" 포트폴리오 웹페이지를 재생성한다
argument-hint: ""
---

# /build-log

아래 단계를 순서대로 수행해 "Build Log" 포트폴리오 웹페이지를 최신 데이터로 재생성한다. Notion·GitHub는 **읽기만** 한다(`/sync-repos`와 역할 분리). CLAUDE.md의 "코드 작성 금지" 제약은 이 명령에 적용되지 않는다 — 이미지 압축·HTML 조립에 Bash/Python을 자유롭게 쓴다. Vercel MCP는 호출하지 않는다(Notion의 `Vercel URL`을 그대로 신뢰).

## 0. 사전 점검
1. Notion 데이터소스 `collection://e47e4d32-3ee6-49c8-a343-1c7ab4f1159a` 접근 확인.
2. GitHub MCP 연결 확인(README 조회용).

## 1. 데이터 수집 및 분류
1. `notion-query-data-sources`를 **view 모드**로 조회: `{"mode": "view", "view_url": "https://app.notion.com/p/by1927/fe68680f5a474f6892ef6d39e3663239?v=53bf797693054bc7b6608a4db91bebd3", "page_size": 100}`. 정렬 없는 뷰라 응답 순서 = 수동 정렬 순서 — **SQL로 재정렬하지 않는다.** 각 행에서 저장소명·Git URL·설명·기술태그·마지막푸시일·활동상태·`Vercel URL`·`포트폴리오 노출`을 읽는다. `has_more`면 `start_cursor`로 이어서 조회.
2. 상태 판정(Vercel API 재조회 없음):
   - 노출 `__NO__` → **완전 제외**(그리드·미배포 목록 모두)
   - 노출 `__YES__` + `Vercel URL` 있음 → **LIVE** (그 값을 Live 링크로. 빌드 실패 상태여도 URL 있으면 그대로 사용, BUILD FAIL 별도 구분 없음)
   - 노출 `__YES__` + `Vercel URL` 없음 → **미배포** (그리드에 안 보이고 최종 보고 집계에만 반영)
3. 카드 순서 = 1번 응답 순서 그대로.

## 2. 재실행 모드 확인
1. 기존 `index.html`에서 카드 데이터(저장소명·설명·기술태그·pushed 날짜·`Vercel URL`)를 추출한다. 파일이 없거나 파싱 실패면 질문 없이 **전체 재생성**으로 진행.
2. 1단계 최신 데이터와 비교한다(저장소 집합 + 필드 값 전부).
3. 완전히 동일하면 "변경 사항이 없습니다. 그래도 전체를 다시 그릴까요?"만 확인 — 아니오면 종료(파일 안 씀).
4. 하나라도 다르면(신규 저장소 유무와 무관하게) 사용자에게 선택하게 한다:
   - **전체 재생성**: LIVE 전체 README·커버 이미지를 다시 가져온다.
   - **증분 업데이트(권장)**: 신규 저장소만 README·커버 이미지를 새로 가져오고, 기존 카드는 필드 값만 최신화(재요청 없음).
5. 두 모드 모두 기술 스택 칩·통계 탭은 전체 데이터로 재계산하고, 3~7단계는 페이지 전체를 다시 조립해서 저장한다 — 생략되는 건 불필요한 README/커버 재조회뿐이다.

## 3. README 수집 (LIVE 전체, 증분 모드는 신규만)
GitHub MCP `get_file_contents`(path: `README.md`) 호출. 404면 "README 없음"(추측 금지). 원문은 6단계에서 JSON으로 임베드(과도하게 긴 README는 핵심만 요약 가능).

**카드 제목**: 저장소명이 아니라 **README 첫 `#` 제목**을 쓴다. 제목에 괄호+영문이 있으면 그 부분만 제거(예: `AI 공감 다이어리 (AI Empathy Diary)` → `AI 공감 다이어리`). 영문만이거나 괄호 없으면 그대로. README 없거나 `#` 없으면 저장소명. 제목이 우연히 겹쳐도 임의로 구분하지 않는다.

## 4. 커버 이미지 (LIVE 전체, 증분 모드는 신규만)
Artifact는 외부 이미지를 CSP로 막으므로 base64로 임베드한다.
1. `https://opengraph.githubassets.com/1/{계정}/{저장소명}` 요청. 429면 몇 초 대기 후 재시도.
2. Pillow: `Image.open(...).convert("RGB")` → `thumbnail((640,640))` → `save(..., format="JPEG", quality=68, optimize=True)`.
3. base64로 `data:image/jpeg;base64,...` 임베드. 대량 base64는 Claude 컨텍스트로 읽지 말고 `__COVER_{TOKEN}__` 플레이스홀더를 Python으로 치환. 증분 모드에서 재사용하는 카드는 기존 base64 값을 그대로 쓴다(재다운로드 없음).

## 5. 폰트
Pretendard 파일을 base64로 임베드하지 않는다(과거 2MB+ 증가 원인). `DESIGN.md`의 시스템 폰트 폴백 스택을 그대로 쓴다.

## 6. 디자인 & 페이지 구성
색상·타이포그래피·아이콘(mask 기반 SVG)·컴포넌트 스타일(칩·토글·카드 호버·배지·모달·통계 바)은 **`DESIGN.md`를 그대로 따른다.** 아래는 이 페이지 전용 콘텐츠 규칙이다.

- 상단: eyebrow `2026 Vibe Coding Study` + h1 `Projects` + lede "카드를 클릭하면 상세 설명을 볼 수 있습니다." (수치·부연 설명 넣지 않음)
- 탭: **포트폴리오** / **통계**. 별도 요약 바 없음 — 숫자는 통계 탭에만.
- 포트폴리오 탭: 기술 스택 필터 칩(빈도 내림차순) → 커버 이미지 토글(**기본 꺼짐**) → 카드 그리드. 미배포 목록은 표시하지 않는다.
- 카드: 커버(2:1) → 저장소명 + LIVE 배지, 설명(있으면), 기술 태그, pushed 날짜 → Live/GitHub 링크 2단(**"GitHub"** 표기, "Code" 금지). 가짜 주소창 요소 없음.
- 모달(카드 클릭 시): **커버 이미지 제외.** 저장소명·배지·설명·태그·Live/GitHub 링크·README 렌더링(자체 마크다운→HTML: 헤딩/굵게/기울임/링크/표/목록/코드펜스, 이미지는 `[이미지: alt]`로 치환). README 없으면 "이 저장소에는 README.md가 없습니다."
- 통계 탭: **`포트폴리오 노출=true` 전체**(배포 여부 무관)를 집계 대상으로 (a) 기술 스택 분포 막대그래프 (b) 활동상태 비율 스택 바+범례(0개도 숫자 표기). 집계 기준은 화면에 표시하지 않는다.

## 7. 산출물 저장
1. **fragment본**: `<title>...</title>`로 시작, `<!doctype>`/`<html>`/`<head>`/`<body>` 없음. 스크래치패드에 저장.
2. Artifact로 발행. 기존 URL이 있으면 그 `url`로 갱신(없으면 `Artifact({action:"list"})`로 찾기). favicon 🚀 고정.
3. **완전한 문서본**: fragment본을 아래로 감싸 프로젝트 루트 `index.html`로 저장(**`<meta charset="UTF-8">` 필수**):
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
4. `index_1.html`이 있어도 건드리지 않는다(무관한 별개 파일).

## 8. 커밋 및 푸시 확인
저장 후 "지금 변경된 `index.html`을 GitHub에 커밋하고 푸시할까요?"라고 먼저 묻는다.
- **예**: 변경된 파일만 스테이징 → 간단한 한국어 커밋 메시지로 커밋 → `git fetch`로 원격 확인(새 커밋 있으면 먼저 merge) → `origin main`에 푸시(강제 push 금지).
- **아니오**: 커밋 없이 로컬 변경만 남기고 종료.

## 최종 보고
| Notion 전체 | LIVE | 미배포 |
|---|---|---|
| N | N | N |

이어서: 실행 모드(전체 재생성/증분/변경 없음, 신규·변경 저장소 목록), Artifact 링크, `index.html` 경로, README 못 찾은 저장소, 커버 이미지 실패 저장소, 커밋·푸시 여부(및 커밋 해시).
