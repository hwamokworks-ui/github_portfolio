# GitHub → Notion 포트폴리오 동기화 실습

GitHub MCP · Notion MCP · Resend MCP 도구 호출과 프롬프트만으로 (코드 작성 없이) GitHub 저장소를 분석해 Notion 포트폴리오 데이터베이스로 정리하고, 완료 이메일을 발송하는 실습 프로젝트다. 여기에 더해, Vercel에 실제로 배포된 저장소만 골라 보여주는 카드형 포트폴리오 페이지도 별도로 만들었다.

## 구성

| 파일 | 내용 |
|---|---|
| `CLAUDE.md` | `/sync-repos` 및 모든 동기화 작업이 따르는 공통 규칙(스키마, 배치 처리, 재시도, 활동상태/포트폴리오 노출 기준 등) |
| `prd.md` | `/sync-repos` 슬래시 명령의 PRD |
| `prd-vercel-portfolio.md` | Vercel 배포 저장소 카드형 포트폴리오(Artifact)의 PRD |
| `.claude/commands/sync-repos.md` | GitHub 저장소 전체를 Notion DB로 동기화하는 슬래시 명령 `/sync-repos` |

## 산출물

- **Notion 데이터베이스**: "GitHub 리포지토리 관리" — 저장소별 기술스택·핵심기능·구조 다이어그램 리포트, 활동상태, 포트폴리오 노출 상위 5개, 대시보드 뷰(갤러리·차트) 포함
- **Vercel 포트폴리오 카드 페이지**: GitHub 계정 저장소 중 `homepage`가 `*.vercel.app`으로 확인된 것만 골라 커버이미지·기술스택·라이브 링크를 보여주는 독립 웹페이지 (Notion과 무관하게 GitHub MCP 데이터만으로 동작)

## 실행 방식

이 프로젝트는 별도 코드베이스가 없다. `/sync-repos` 슬래시 명령을 실행하면 Claude Code가 CLAUDE.md 규칙에 따라 GitHub·Notion·Resend MCP를 순서대로 호출해 전 과정을 자동으로 수행한다.
