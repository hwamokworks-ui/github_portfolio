# DESIGN.md — Build Log 디자인 시스템

이 문서는 `index.html`(Build Log 포트폴리오 페이지)에서 쓰인 디자인을 다른 프로젝트에도 그대로 적용할 수 있도록 토큰·타이포그래피·컴포넌트 스펙·인터랙션 규칙으로 정리한 것입니다. 색상 값과 치수를 그대로 복사해서 쓰거나, 브랜드에 맞게 토큰 값만 교체해서 재사용하세요.

## 1. 색상 (라이트 테마 고정)

다크모드 자동 전환은 쓰지 않습니다. CSS 커스텀 프로퍼티로 선언해두면 필요 시 다크 테마를 별도로 얹기도 쉽습니다.

```css
:root{
  --paper: #f2f0ea;   /* 페이지 배경 */
  --card:  #ffffff;   /* 카드/모달 배경 */

  --ink:       #1e1c22; /* 본문 텍스트, 제목 */
  --ink-dim:   #625f68; /* 보조 텍스트, 설명 */
  --ink-faint: #9b98a1; /* 최소 강조 텍스트 (날짜, 카운트) */

  --line:      #dedad2; /* 일반 구분선, 트랙 배경 */
  --card-line: #c3bcae; /* 카드 전용 테두리 — line보다 한 톤 짙게 */

  --accent: #4b3f72;    /* 브랜드 강조색 (보라) — 활성 상태, 링크, 포커스 */

  --live:    #3f7a5c;  /* 성공/활성 상태 */
  --live-bg: #e6efe8;

  --error:    #a15c3e; /* 에러/경고 상태 */
  --error-bg: #f5e9e1;
}
```

**원칙**
- 배경은 순백이 아니라 살짝 웜톤인 `--paper`(#f2f0ea)를 쓰고, 그 위에 얹는 카드만 순백(`--card`)으로 띄워 입체감을 준다.
- 텍스트는 검정 대신 `--ink`(다크 그레이 계열)로, 3단계(ink / ink-dim / ink-faint)로만 위계를 나눈다 — 4단계 이상 늘리지 않는다.
- 테두리는 두 가지뿐: 일반 구분선(`--line`)과 카드 전용 테두리(`--card-line`, 더 진함). 카드가 배경과 잘 구분되도록 카드 테두리를 한 톤 더 진하게 쓰는 것이 핵심이다.
- 강조색(`--accent`)은 보라 하나로 통일 — 활성 탭 밑줄, 선택된 칩, 링크, 토글 On 색 전부 동일한 값을 쓴다. 색을 늘리지 않고 하나의 강조색을 반복 사용해야 통일감이 생긴다.

## 2. 타이포그래피

```css
body{
  font-family:'Pretendard', -apple-system, BlinkMacSystemFont, "Segoe UI", "Apple SD Gothic Neo", sans-serif;
}
```

- 본문 전체(숫자, 라벨, 칩, 배지, 태그 포함) 를 예외 없이 하나의 산세리프 폰트로 통일한다. `Pretendard`는 웹폰트 파일을 따로 임베드하지 않아도 되며, `@font-face` 선언이 없으면 폴백 스택(시스템 폰트)으로 자연스럽게 렌더링된다 — 폰트 파일 무게 없이 한글 프로젝트에서 무난한 형태를 얻는 방법이다.
- 코드/README 안의 실제 코드 스니펫(`<code>`, `<pre>`)만 예외로 모노스페이스(`ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, monospace`)를 쓴다. 그 외 어디에도 monospace나 serif를 섞지 않는다.

| 요소 | 크기 | 굵기 | 비고 |
|---|---|---|---|
| h1 (페이지 제목) | 40px | 700 | `letter-spacing:-0.01em`, `line-height:1.15` |
| eyebrow (영문 라벨) | 12.5px | 600 | 대문자 변환, `letter-spacing:.12em`, accent 색 |
| lede (부제 한 문장) | 15.5px | 400 | `--ink-dim`, `max-width:640px`로 줄 길이 제한 |
| 탭 버튼 | 14.5px | 600 | 비활성은 `--ink-faint`, 활성은 `--ink` + accent 밑줄 |
| 카드 제목 | 16px | 700 | `line-height:1.35` |
| 카드 설명 | 13.5px | 400 | `--ink-dim`, `line-height:1.5` |
| 태그/배지 | 11~11.5px | 500~700 | 배지는 `letter-spacing:.04em` |
| 모달 제목 | 22px | 700 | |
| README 렌더링 헤딩 | 16.5px 고정 | 700 | 원본 마크다운의 h1~h4를 전부 같은 크기로 눌러서, 모달 안에서 헤딩이 과도하게 커 보이지 않게 한다 |

작은 화면(≤640px)에서는 h1만 30px로 줄인다. 그 외 요소는 고정 크기를 유지해도 무방할 만큼 여유 있게 잡혀 있다.

## 3. 레이아웃

```css
.wrap{ max-width:1140px; margin:0 auto; padding:56px 24px 96px; }
```

- 콘텐츠 폭은 1140px로 제한하고 중앙 정렬, 상하 여백을 넉넉히(56px/96px) 준다.
- 섹션 간 수직 리듬: h1 아래 14px → lede 아래 36px → 탭 아래 28px → 칩 줄과 컨트롤 줄 사이 20px → 컨트롤 줄 아래 22px. 딱 떨어지는 숫자보다 "다음 블록과의 시각적 무게"를 보고 6~8px 단위로 미세 조정한 값들이다.
- 카드 그리드: `grid-template-columns: repeat(auto-fill, minmax(260px,1fr))`, `gap:22px`. 컨테이너 폭에 따라 카드 개수가 자동으로 줄었다 늘었다 하도록 고정 컬럼 수를 쓰지 않는다.

## 4. 컴포넌트 스펙

### 탭 (Tabs)
- 밑줄 스타일: 컨테이너 하단에 1px `--line` 보더, 활성 탭에는 `::after`로 2px accent 밑줄을 그린다.
- 배경색 변화나 필박스(pill) 없이 텍스트 색 + 밑줄만으로 상태를 표현한다.

### 필터 칩 (Chip)
```css
.chip{
  border:1px solid var(--card-line); background:var(--card); color:var(--ink-dim);
  border-radius:999px; padding:6px 14px; font-size:13px; font-weight:500;
}
.chip:hover{ border-color:var(--accent); }
.chip.active{ background:var(--accent); border-color:var(--accent); color:#fff; }
```
- 기본 상태는 흰 배경 + 연한 텍스트, **호버는 테두리만 accent로 바뀌는 정도로 절제**한다 (배경을 채우지 않는다 — 배경 채움은 active 전용).
- "전체" 같은 초기화 칩을 항상 포함해, 아무 조건도 선택하지 않은 기본 상태를 명시적으로 보여준다. 개별 칩을 하나라도 선택하면 "전체"는 비활성으로, 전부 해제하면 다시 "전체"가 활성으로 돌아오게 동기화한다.
- 칩 뒤에 개수 배지(`<span class="count">`)를 붙여 필터링 전 몇 건이 해당하는지 미리 알려준다.

### 토글 스위치 (Switch)
```css
.switch{ width:38px; height:22px; }
.slider{ background:var(--line); border-radius:999px; }
.slider::before{ width:16px; height:16px; background:#fff; box-shadow:0 1px 2px rgba(0,0,0,.25); }
.switch input:checked + .slider{ background:var(--accent); }
.switch input:checked + .slider::before{ transform:translateX(16px); }
```
- 네이티브 체크박스를 시각적으로 숨기고 `<span class="slider">`로 트랙/노브를 그리는 표준 패턴. 라벨 텍스트는 토글보다 먼저(왼쪽) 배치해 무엇을 켜고 끄는지 읽기 쉽게 한다.
- 기본값이 꺼짐인 토글은 "부가 정보를 숨겨서 화면을 가볍게 유지"하는 용도로만 쓴다 (예: 커버 이미지 표시 여부).

### 카드 (Card)
```css
.card{ border:1px solid var(--card-line); border-radius:14px; }
.card:hover{
  transform:translateY(-3px);
  box-shadow:0 0 0 1px var(--accent), 0 14px 28px -14px rgba(30,28,34,.28);
}
```
- 호버 시 **테두리 두께 자체는 바꾸지 않는다** — `box-shadow`의 첫 번째 값(`0 0 0 1px`)으로 테두리 위에 accent 컬러 링을 얹어 "두꺼워 보이는" 효과만 낸다. `border-width`를 직접 바꾸면 주변 레이아웃이 1px씩 밀리는 문제가 생긴다.
- 카드 전체가 클릭 가능한 하나의 요소여야 하고, 내부의 개별 링크(Live/GitHub)는 `event.stopPropagation()`으로 카드 클릭과 분리한다.
- 카드 구성 순서: 커버 이미지(2:1, `object-fit:cover`, 토글로 표시/숨김) → 본문(제목+상태 배지 → 설명 → 태그 → 날짜) → 하단 액션 2단 그리드.
- 하단 두 버튼(Live/GitHub)은 **완전히 동일한 스타일**을 쓴다. 한쪽만 배경을 채우는 강조 버튼으로 만들지 않는다 — 배경을 채우면 `currentColor` 기반 아이콘이 배경과 같은 계열이 되어 아이콘이 안 보이는 문제가 생길 수 있다.

### 배지 (Badge)
```css
.badge{ border-radius:999px; padding:3px 9px; font-size:11px; font-weight:700; letter-spacing:.04em; }
```
- 상태별 배경/텍스트 컬러 쌍을 미리 정의해둔다 (`--live`/`--live-bg`, `--error`/`--error-bg`). 배지 색만 보고도 상태를 구분할 수 있어야 한다.

### 모달 (Modal)
- 오버레이: `rgba(30,28,34,.5)` 반투명 배경, `align-items:flex-start`로 위쪽 정렬해 긴 콘텐츠(README)도 스크롤 자연스럽게 시작되게 한다.
- 모달 카드는 `max-width:640px`로 본문 가독폭을 제한.
- 닫기 버튼은 원형 아이콘 버튼(30×30px), 우상단 고정.
- 모달 안에서는 커버 이미지나 과도한 장식 없이 텍스트 정보(제목/배지/설명/태그/링크/본문) 위주로 구성한다.

### 통계 바 (Stat bars)
- 단순 막대그래프: 최댓값 대비 비율(%)로 `bar-fill` 폭을 계산해 라벨-트랙-카운트 3열 그리드로 배치.
- 비율/구성 데이터는 스택 바(stacked bar) + 범례 조합으로: 각 세그먼트 폭을 퍼센트로 나누고, 0%인 항목도 숫자를 반드시 텍스트로 표기한다 (막대에서 안 보인다고 생략하지 않는다).

## 5. 아이콘

유니코드 글리프(↗, ✕ 등)를 쓰지 않는다 — 폰트/OS에 따라 다르게 렌더링되거나 깨질 수 있다. 대신 `currentColor` 기반 SVG를 **CSS `mask`** 로 적용해 배경색이 바뀌어도 아이콘 색이 텍스트 색을 자동으로 따라가게 한다.

```css
.icon{ width:13px; height:13px; background:currentColor; }
.icon-arrow{
  mask:url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="black" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"><line x1="7" y1="17" x2="17" y2="7"/><polyline points="8 7 17 7 17 16"/></svg>') center/contain no-repeat;
}
```
- data URI 안의 `stroke="black"`/`fill="black"` 값은 실제로 렌더링되지 않는다 (mask는 알파 채널만 사용) — `background:currentColor`가 실제 색을 결정한다.
- 외부 아이콘 폰트나 라이브러리 없이 인라인 SVG만으로 해결되므로 의존성이 늘지 않는다.

## 6. 인터랙션 규칙 요약

- **전환은 짧고 균일하게**: 호버·토글 전환은 `.15s`, 카드 호버는 `.18s`. 전부 `ease`.
- **강조는 accent 하나로**: 색을 여러 개 쓰지 않고, "선택됨/활성/강조"는 전부 같은 accent 컬러로 통일한다.
- **레이아웃을 흔들지 않는 강조 기법**: 테두리 두께 대신 `box-shadow` 링, 폭 변화 대신 `transform`을 쓴다.
- **기본값은 절제된 쪽으로**: 부가 정보(커버 이미지 등)는 기본 숨김, 필요할 때만 토글로 보여준다.
- **없음/0도 명시적으로 보여준다**: 빈 상태 문구, 0건인 통계 항목도 숫자로 그대로 노출한다 — 생략하면 "집계가 안 된 것"처럼 보인다.

## 7. 적용 방법

1. 위 `:root` 토큰 블록을 프로젝트에 그대로 복사한다.
2. 브랜드 컬러가 다르면 `--accent`(그리고 필요하면 `--live`/`--error` 페어)만 교체한다 — 나머지 그레이스케일 토큰(`--paper`/`--ink`/`--line` 계열)은 대부분의 프로젝트에 무난하게 어울린다.
3. 폰트를 다른 걸로 바꾸더라도 "본문 전체 단일 폰트 + 코드만 모노스페이스" 원칙은 유지한다.
4. 컴포넌트는 위 스펙을 그대로 가져다 쓰거나, 칩/토글/카드/모달 중 필요한 것만 골라 이식해도 무방하다 — 서로 독립적으로 설계되어 있다.
