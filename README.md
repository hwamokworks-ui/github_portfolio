# GitHub → Notion 포트폴리오 동기화 실습

GitHub MCP · Notion MCP · Vercel MCP · Resend MCP 도구 호출과 프롬프트만으로 (코드 작성 없이) GitHub 저장소 전체를 분석해 Notion 포트폴리오 데이터베이스로 정리하고, 완료 이메일을 발송하는 실습 프로젝트다. 여기에 더해, 그 Notion 데이터를 그대로 재료 삼아 카드형 포트폴리오 웹페이지("Build Log")를 만드는 두 번째 슬래시 명령도 있다.

## 구성

| 파일 | 내용 |
|---|---|
| `CLAUDE.md` | `/sync-repos` 및 모든 동기화 작업이 따르는 공통 규칙(스키마, 배치 처리, 재시도, 활동상태/포트폴리오 노출 기준, Vercel URL 매칭 등) |
| `prd.md` | `/sync-repos` 슬래시 명령의 PRD |
| `prd-vercel-portfolio.md` | 카드형 포트폴리오 웹페이지("Build Log", `/build-log`)의 PRD |
| `.claude/commands/sync-repos.md` | GitHub 저장소 전체를 Notion DB로 동기화하는 슬래시 명령 `/sync-repos` |
| `.claude/commands/build-log.md` | Notion·GitHub 최신 상태로 카드형 포트폴리오 웹페이지를 재생성하는 슬래시 명령 `/build-log` |

## 산출물

- **Notion 데이터베이스**: "GitHub 리포지토리 관리" — 저장소별 기술스택·핵심기능·구조 다이어그램 리포트, 활동상태, Vercel 실배포 주소, 대시보드 뷰(갤러리·차트) 포함. `포트폴리오 노출` 여부와 카드 정렬 순서는 사용자가 Notion에서 직접 정한 값을 그대로 유지하며(재실행이 되돌리지 않음), 새로 발견되는 저장소만 미노출 상태로 맨 끝에 추가된다
- **"Build Log" 포트폴리오 카드 페이지**: 위 Notion DB에서 `포트폴리오 노출`이 켜져 있고 `Vercel URL`이 채워진 저장소만 골라, 지정된 Notion 뷰의 수동 정렬 순서 그대로 커버이미지·기술스택·README 미리보기·라이브 링크를 보여주는 웹페이지. `/build-log` 실행 시 공유용 Artifact와, 실제 Vercel 배포가 가능한 가벼운 `index.html`(프로젝트 루트) 두 가지를 함께 만든다

## 실행 방식

이 프로젝트는 별도 코드베이스가 없다. `/sync-repos`를 실행하면 Claude Code가 CLAUDE.md 규칙에 따라 GitHub·Notion·Vercel·Resend MCP를 순서대로 호출해 Notion DB 동기화 전 과정을 자동으로 수행한다. 그다음 `/build-log`를 실행하면 그 Notion DB와 GitHub README를 읽기만 해서(쓰기 없음) 카드형 포트폴리오 페이지를 최신 데이터로 재생성한다.
