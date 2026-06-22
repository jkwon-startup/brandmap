# 개발문서 — 여행가J의 브랜드맵 워크시트

> 수강생이 6단계 질문에 답하면 **브랜드맵 마크다운 문서**가 자동 생성되는 단일 HTML 웹앱.
> 자매 프로젝트 `CEO_worksheet`의 config 단일 소스 아키텍처를 그대로 복제하고,
> 콘텐츠(`WS` 배열)와 출력 직렬화(`buildDoc`/`serializeBlock`)만 브랜드맵 구조로 교체했다.

- **결과물**: `index.html` 단일 파일 (빌드 단계 · 백엔드 · 프레임워크 없음)
- **라이브**: https://jkwon-startup.github.io/brandmap/
- **참고 아키텍처**: `/Users/kwonjungsun/DEV/CEO_worksheet/index.html` (+ 그 `DEVELOPMENT.md`)
- **출력 포맷 기준**: `브랜드맵 최종 출력 예시/브랜드 맵 예시 .md` (Student Ver. 템플릿)
- 작성일: 2026-06-22

---

## 1. 무엇을 만들었나

- 6개 PART 탭(브랜드 정의 · 고객 · 시장 · 표현 · 실행 · 미래) / 총 19개 섹션
- 각 입력칸에 **작성 가이드(`sub`) + 예시(placeholder)** 동시 표시 → 초보 수강생도 막힘없이 작성
- 입력 즉시 **localStorage 자동 저장** (새로고침해도 유지)
- PART별 **완성 체크리스트로 진행률 추적** (N/6)
- **마크다운 · 텍스트 · 인쇄(PDF)** 다운로드 — 출력은 브랜드맵 예시와 동일한 구조
- **새 브랜드맵** 버튼으로 전체 초기화 / 헤더에 카카오 오픈채팅 버튼

---

## 2. 기술 스택 & 설계 원칙

CEO_worksheet와 동일: 단일 `index.html`(인라인 CSS+JS) · 바닐라 JS · CDN 폰트만 · localStorage · GitHub Pages.

**핵심 원칙: config 단일 소스.** 워크시트 전체를 `WS` 배열로 정의하고 렌더링 · 저장 · 다운로드가
모두 이 배열에서 파생된다. 콘텐츠를 바꿀 때 HTML을 손대지 않고 `WS`만 수정.

디자인 토큰(`:root`)/폰트(Noto Serif KR · Pretendard)/블록 렌더 엔진은 CEO_worksheet에서 그대로 차용.

---

## 3. 파일 구조

```
brandmap_sheet/
├─ index.html               # 전부 (HTML+CSS+JS)
├─ README.md
├─ DEVELOPMENT.md           # (이 문서)
├─ .gitignore               # .DS_Store, *.log, 출력예시 폴더 제외
└─ 브랜드맵 최종 출력 예시/   # 출력 포맷 기준(로컬 전용, 배포 제외)
```

---

## 4. ⭐ CEO_worksheet와 다른 점 (이 프로젝트의 핵심 차이)

브랜드맵은 "여러 워크시트"가 아니라 **하나의 브랜드맵 문서**가 출력 목표다.
따라서 출력 직렬화만 다음과 같이 조정했다.

### 4.1 WS 객체에 추가된 필드
- `ws.part` — 출력 시 `## PART n. ...` 헤딩으로 사용 (UI의 `ws.title`과 별도).
- `ws.n` = `'1'`~`'6'`, `ws.time` = 섹션 요약 칩.

### 4.2 블록의 출력 제어
- `b.skipDoc: true` — 출력 문서에서 제외 (브랜드 이름 입력 `bm_name`이 사용. 제목으로만 쓰임).
- `type:'checklist'` 블록은 `buildDoc`에서 자동 제외 (작성용 보조 장치).

### 4.3 헤딩 스타일 (`serializeBlock`)
- 마크다운 출력에서 블록 라벨을 `**라벨**`이 아니라 **`### 라벨`**(섹션 헤딩)으로 emit.
- `fields` → `* **필드명:** 값` (템플릿의 불릿 형식과 일치).
- `table` → 마크다운 표, `text` → 본문 문단, `tags` → `값 / 값 / 값`.

### 4.4 문서 제목 (`buildDoc`)
- 첫 줄 `# {bm_name 값} 브랜드 맵`. 빈 값이면 `(브랜드명)`.
- 각 PART는 `## {ws.part}`, 그 아래 섹션은 `### {block.label}`.
- 빈 입력은 `(작성 전)`/`_(작성 전)_`로 표기 (예시 placeholder가 출력에 새지 않음 — `val(key)`만 직렬화).

### 4.5 파일명
`{브랜드명}_브랜드맵_{YYYY-MM-DD}.md|txt` (브랜드명에서 파일명 금지문자 제거).

> 그 외 렌더 엔진(`renderBlock`/`inputCell`/`renderOptions`/`renderField`), 저장/복원(`saveData`/`restoreData`),
> 탭/진행률(`switchTab`/`updateProgress`), 리셋/임포트는 CEO_worksheet와 동일.

---

## 5. data-key 네이밍 규약

`bm_{영역}_{항목}` — 예: `bm_purpose`, `bm_val1_name`, `bm_persona1_desc`, `bm_pdr_problem`, `bm_road0_mvp`.
체크리스트는 `bm_p{n}_chk{m}`. **키를 바꾸면 기존 사용자 저장값과 끊긴다(키 유지 권장).**
localStorage 키: `brandmap_sheet_v1` / `brandmap_sheet_v1_tab` (CEO와 분리).

---

## 6. 변경/확장 방법

- **섹션 추가/수정**: 해당 PART의 `blocks`에 블록 객체 추가. 새 `data-key`만 부여하면 저장/출력 자동 반영.
- **PART 추가/삭제**: `WS` 배열에 객체 추가/제거. 탭 · 진행률(`N/M`) 자동 보정.
- **새 블록 타입**: `renderBlock()`과 `serializeBlock()`에 동일 `case` 추가(이 둘만 맞추면 됨).
- **디자인 톤 변경**: `:root` 변수만 수정.

---

## 7. 배포 (GitHub Pages)

```bash
cd /Users/kwonjungsun/DEV/brandmap_sheet
git init -b main
git add .gitignore index.html README.md DEVELOPMENT.md
git commit -m "init: 브랜드맵 워크시트 웹앱"
gh repo create brandmap --public --source=. --remote=origin --push
gh api -X POST repos/jkwon-startup/brandmap/pages -f "source[branch]=main" -f "source[path]=/"
```
URL: `https://jkwon-startup.github.io/brandmap/`. 첫 빌드 1~2분.

### 갱신
```bash
git add index.html && git commit -m "..." && git push
```

### ⚠️ 빌드 큐 충돌 주의
짧은 간격으로 여러 번 push 하거나 빌드 진행 중 수동 재빌드를 또 호출하면 `errored` 충돌 발생.
→ "진행 중 빌드 종료 대기 → 그 다음 1회만 트리거". 라이브 확인은 `?_=n` 캐시버스터로.

---

## 8. 검증 (배포 전, file:// 로컬)

CEO_worksheet `DEVELOPMENT.md` §12의 Playwright headless 스니펫 재사용. 표준 항목:
1. 렌더: 6개 탭 + 19개 섹션 헤딩 + 브랜드명 입력 존재, 콘솔에러 0.
2. 자동저장: 입력 → 800ms 후 `localStorage['brandmap_sheet_v1']` 반영.
3. 복원: reload 후 입력값 유지.
4. 출력: `buildDoc('md')`가 `# ... 브랜드 맵` + `## PART 1`~`## PART 6` + `### 1`~`### 19` + 표를 포함.
5. 진행률: 한 PART의 checklist 전체 체크 시 `N/6` 증가.
6. 데스크톱/모바일 스크린샷.
