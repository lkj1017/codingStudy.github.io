# 티스토리 수익형 콘텐츠 생성 에이전트 계획서 (v3)

- 대상 블로그: lifegoals001.com (티스토리, 애드센스 승인 완료)
- 문체 후처리: [epoko77-ai/im-not-ai](https://github.com/epoko77-ai/im-not-ai) `humanize-korean` v2.3.2 (MIT)
- 실행 환경: Claude Code (im-not-ai가 Claude Code 플러그인이라 같은 런타임으로 묶는다)
- 작성일: 2026-10-24 / v2·v3 개정: 2026-10-24, 2026-10-25

개정 이력

| 버전 | 바뀐 것 |
|---|---|
| v2 | 4장: 단계마다 담당 에이전트·스킬·도구·전문성 장치 지정, 썸네일·본문 이미지 단계 추가 / 6장: 애드센스 금지·제한 주제 하드 필터, 구글 권장 SEO 구조와 마크업 규칙 / 7장: A방식 확정 / 8장: 실행 위치 추천안 / 9장: 처음부터 Search Console API |
| v3 | 6.2: 주력 카테고리 확정(생활 돈 관리·제도 활용 60%, 목표 달성·자기계발 도구 40%)과 주제 허브·시즌 캘린더 / 4.7: 네이버 검색광고·데이터랩·Gemini API 준비 절차 신설 / 이미지 생성은 Google AI Pro 크레딧으로 Gemini API 사용 / 쿠팡 파트너스 등 제휴는 하지 않음 / 저장소는 새 private GitHub 저장소 / 13장: 결정 내역과 0주차 준비물 |

---

## 0. 한눈에 보기

키워드를 고르면 에이전트가 정책 검사, 검색 의도 분석, 리서치, 초안, 사실 검증, im-not-ai 윤문, 썸네일·본문 이미지, 구글 권장 구조의 HTML까지 만들어 검토 대기함에 넣는다. 사람은 글 하나에 15분 안팎을 쓴다. 경험과 사진을 보태고, 미리보기에서 HTML을 복사해 티스토리에 붙여넣고, 이미지를 올린 뒤 예약 발행한다.

완전 무인 발행으로 설계하지 않은 이유는 그대로다.

1. 티스토리 공식 Open API가 2024년 2월에 종료돼 정식 자동 발행 경로가 없다.
2. 구글은 AI 글 자체를 막지 않지만, 가치를 보태지 않은 글을 대량으로 찍어내면 스팸 정책(scaled content abuse)과 애드센스 저가치 콘텐츠 정책에 걸린다.

im-not-ai는 사실·수치는 그대로 두고 번역투와 AI 관용구만 걷어내는 윤문기다. 글을 새로 쓰지 않으므로 "사실 검증과 정책 검수가 끝난 뒤, HTML로 바꾸기 전"에 둔다.

---

## 1. 확인된 전제와 제약

| 항목 | 확인 내용 | 계획에 주는 영향 |
|---|---|---|
| 티스토리 API | 2024년 2월 Open API 글쓰기 종료 | A방식(HTML 붙여넣기)으로 발행 |
| 구글 스팸 정책 | "대량 + 낮은 부가가치 + 순위 조작 목적"을 본다. 기준 편수는 비공개 | 하루 1편, 글마다 1차 정보 1개 이상, 사람 승인 |
| 애드센스 정책 | 게시자 정책(금지 콘텐츠) 위반은 광고 게재 중지·계정 해지로 이어질 수 있다. 게시자 제한사항은 위반은 아니지만 광고가 제한돼 수익이 급감한다 | 두 범주 모두 주제 단계에서 하드 차단 (6.1) |
| FAQ·HowTo 리치 결과 | 2023년 8월부터 FAQ 리치 결과는 공신력 있는 정부·보건 사이트에만 노출, HowTo는 중단 | FAQ는 독자를 위해 본문에 두되, 리치 결과를 기대하고 마크업에 공들이지 않는다 |
| im-not-ai | 10개 카테고리 85개 패턴, `장르: 블로그` 지원, 변경률 30% 경고·50% 중단, `verify_gates.py`가 수치·고유명사 보존을 코드로 검사 | ⑧ 윤문 단계에 배치 |
| Search Console API | 검색 실적 조회 사이트당 분당 1,200회, URL 검사 사이트당 하루 2,000회·분당 600회 | 개인 블로그 규모에서는 한도 걱정 없이 매일 동기화 가능 |
| 클라우드 네트워크 (이번 세션 실측) | `googleapis.com` 계열 통과. `gov.kr`·`nts.go.kr`·`bokjiro.go.kr`·`openapi.naver.com`·`tistory.com`·`lifegoals001.com` 차단 | 클라우드로 옮길 때 네트워크 허용 범위 변경 필요 (8장) |
| 블로그 현황 | 위 차단 때문에 블로그를 직접 보지 못했고, 검색 색인에서도 글이 거의 잡히지 않음 | 새로 키우는 블로그로 보고 경쟁이 약한 긴 키워드 위주로 주력 카테고리 선정 (6.2) |

---

## 2. 목표와 KPI

수익은 대략 `월 페이지뷰 × 페이지 RPM ÷ 1000`이다.

| 시점 | 지표 | 목표 | 측정 |
|---|---|---|---|
| 1개월 | 발행 편수 / 글당 사람 작업 시간 | 15~20편 / 15분 이하 | `published.json`, 작업 로그 |
| 1개월 | 색인율 | 발행 14일 안에 80% 이상 색인 | URL Inspection API |
| 3개월 | 노출·클릭, 10위 안 쿼리 수 | 월별 우상향, 10위 안 쿼리 20개 이상 | Search Analytics API |
| 3개월 | 카테고리별 RPM | 상위 카테고리 비중 확대 | 애드센스 리포트 |
| 상시 | 품질·정책 | im-not-ai 등급 A/B 90% 이상, 정책 경고 0건, 사실 오류 신고 0건 | `qa.json` |

---

## 3. 전체 구조

```
[주 1회] /plan-week
  Search Console API + 네이버 키워드 데이터 ─▶ 주제 후보 ─▶ 애드센스 정책 필터 ─▶ backlog.csv (사람 승인)
                                                                            │
[하루 1편] /new-post "키워드"                                                 ▼
  ① 브리프·검색 의도 ─▶ ② 리서치 ─▶ ③ 경험·사진 입력(사람) ─▶ ④ SEO 아웃라인 ─▶ ⑤ 초안
                                                                              │
  ⑨ 이미지 제작 ◀──────────────┐                                              ▼
       │              ⑧ im-not-ai 윤문 ◀─ ⑦ 정책 검수 ◀─ ⑥ 사실 검증 ◀────────┘
       ▼                       │
  ⑩ SEO 마크업·HTML ◀──────────┘
       │
  ⑪ QA 게이트 (코드 검사)
       │
  검토 대기함 ─▶ ⑫ 사람 검토 + 붙여넣기 발행(A방식) ─▶ ⑬ 사후 처리 (URL 검사 D+3·7·14)

[매일] gsc_sync.py ─▶ data/gsc.sqlite        [주 1회] /weekly-report ─▶ 리라이트·새 주제 제안
```

⑨ 이미지 제작은 ⑦ 정책 검수가 끝나면 ⑧ 윤문과 병렬로 돌릴 수 있다. 헤딩 구조는 윤문에서도 유지되므로 이미지 위치가 바뀌지 않는다.

---

## 4. 파이프라인 단계별 설계

### 4.1 구성 요소 지도

| 종류 | 이름 | 역할 | 쓰는 단계 |
|---|---|---|---|
| 외부 스킬 | im-not-ai `humanize-korean` | AI 문체 윤문 + 결정적 게이트 | ⑧ |
| 외부 스킬 | im-not-ai `humanize-redo` / `humanize-scan` | 부분 재윤문 / 문제 진단용 빠른 점검 | ⑧ 재시도, 디버깅 |
| 개발 도구 스킬 | Anthropic `skill-creator` | 아래 커스텀 스킬을 만들고 평가(eval) 돌리기 | 구축 단계 |
| 커스텀 스킬 | `/plan-week` | 주간 주제 기획 오케스트레이터 | ⓪ |
| 커스텀 스킬 | `/new-post` | 글 한 편 오케스트레이터 (①~⑪) | ①~⑪ |
| 커스텀 스킬 | `adsense-policy-check` | 정책 판정 규칙·키워드 사전·판정 형식 | ⓪ ① ⑦ ⑪ |
| 커스텀 스킬 | `seo-structure` | 구글 권장 구조, 유형별 아웃라인, 마크업 규칙 | ④ ⑩ |
| 커스텀 스킬 | `image-studio` | 이미지 기획서 형식, 템플릿 렌더링, 최적화 규칙 | ⑨ |
| 커스텀 스킬 | `tistory-format` | 마크다운 → 티스토리 HTML, 미리보기 생성 | ⑩ ⑫ |
| 커스텀 스킬 | `gsc-insights` | Search Console 데이터 질의·분석 레시피 | ⓪ ① ⑬, 주간 리포트 |
| 커스텀 스킬 | `/refresh-post`, `/weekly-report` | 기존 글 보강, 주간 성과 분석 | 9장 |
| 서브에이전트 | 8종 (4.2) | 단계별 전문 작업 | ⓪~⑩ |
| 스크립트 | Python (5장 `scripts/`) | API 호출, 렌더링, 검사처럼 정확해야 하는 일 | 전 단계 |
| 외부 API | Search Console API | 검색 실적, URL 색인 검사, 사이트맵 | ⓪ ① ⑬ 9장 |
| 외부 API | 네이버 검색광고 API (키워드 도구) | 한국어 키워드 월간 검색량·경쟁 정도 | ⓪ |
| 외부 API | 네이버 데이터랩 검색어 트렌드 API | 계절성·추세 | ⓪ |
| 외부 API | Gemini API 이미지 모델 (Google AI Pro 월 크레딧 사용) | 개념 일러스트 (글당 0~1장) | ⑨ |
| 외부 API (선택) | PageSpeed Insights API | 월 1회 대표 글 Core Web Vitals 점검 | 9장 |
| MCP (선택) | Figma | 썸네일 템플릿 시안 디자인, 필요 시 일러스트 생성(Figma AI 크레딧 차감) | 구축, ⑨ |
| 기본 도구 | WebSearch, WebFetch, Bash, Playwright, Pillow, matplotlib | 검색, 원문 확인, 렌더링·캡처, 이미지 처리, 차트 | 전 단계 |

원칙: 판단은 에이전트가, 정확해야 하는 계산·검사·렌더링은 스크립트가 한다. 에이전트 출력은 항상 정해진 마크다운·JSON 형식으로 받아서 다음 단계가 기계적으로 읽게 한다.

### 4.2 서브에이전트 정의 (`.claude/agents/`)

| 에이전트 | 모델 | 허용 도구 | 참조 문서 | 핵심 산출물 |
|---|---|---|---|---|
| `topic-strategist` | sonnet | Read, Write, Bash(키워드·GSC 스크립트), WebSearch | `blog.yaml`, `adsense-topic-policy.yaml`, `gsc-insights` | 주제 후보표 (점수·근거·정책 판정·경험 질문) |
| `serp-analyst` | sonnet | WebSearch, WebFetch, Read, Write | `seo-structure` | 검색 의도 분류, 상위 5개 글 구조·공통 항목·빠진 정보 |
| `researcher` | sonnet | WebSearch, WebFetch, Read, Write | `source-tiers.md` | 사실 목록 (주장·값·출처 URL·출처 등급·확인일) |
| `writer` | opus | Read, Write | `style-guide.md`, `templates/`, im-not-ai `quick-rules.md`, `seo-structure` | 아웃라인, 초안 |
| `fact-checker` | opus | Read, Write, WebFetch | `01_research.md` | 주장 단위 대조표 (일치/불일치/근거 없음) + 수정 초안 |
| `policy-reviewer` | sonnet | Read, Write | `adsense-topic-policy.yaml`, `policy-notes.md` | BLOCK/AVOID/CAUTION/ALLOW 판정 + 근거 문장 |
| `visual-designer` | sonnet | Read, Write, Bash(렌더링 스크립트), Figma MCP(선택) | `image-guidelines.md`, `image-studio` | 이미지 기획서 (위치·유형·문구·alt·캡션) |
| `seo-editor` | sonnet | Read, Write, Bash | `seo-structure`, `published.json`, `gsc.sqlite` | 제목 3안, 내부 링크, `meta.json` |

모델 배정은 출발점이다. 글 품질에 직결되는 `writer`·`fact-checker`만 상위 모델을 쓰고, 1주차 실측 후 조정한다. 각 에이전트 정의에는 "하지 말 것"을 명시한다. 예를 들어 `writer`는 `02_experience.md`에 없는 경험을 만들지 않고, `researcher`는 3등급 출처(개인 블로그·커뮤니티)를 사실 근거로 쓰지 않는다.

### 4.3 단계별 상세

#### ⓪ 주제 기획 — `/plan-week` (주 1회)

- 담당: `topic-strategist` + `adsense-policy-check` + `gsc-insights`
- 도구: `gsc_sync.py`(Search Console API), `naver_keywords.py`(검색광고 API 키워드 도구), `naver_trend.py`(데이터랩), WebSearch
- 전문성 장치
  - 정책 필터를 점수 계산보다 먼저 돌린다. BLOCK·AVOID 판정은 그 자리에서 탈락 (6.1)
  - 우선순위 점수 = 수요(월간 검색량) × 단가 등급(카테고리별, 나중엔 자체 RPM으로 보정) × 블로그 적합도 × 경험 보유 여부 ÷ 경쟁도
  - Search Console에서 "노출은 있는데 전용 글이 없는 쿼리"를 우선 후보로 올린다
  - 시즌 주제는 검색량이 오르기 2~4주 전에 배치 (데이터랩 추세로 판단)
- 산출물: `backlog.csv` 갱신안, 주제별 경험 질문 3개, 필요한 사진·스크린샷 목록
- 통과 조건: 정책 판정 ALLOW 또는 CAUTION, 기존 글과 겹치지 않음, 사람 승인

#### ① 브리프·검색 의도 분석

- 담당: `serp-analyst` + `gsc-insights`
- 도구: WebSearch(상위 결과), WebFetch(상위 5개 글 구조 확인), `gsc_query.py`
- 전문성 장치
  - 검색 의도를 정보형·방법형·비교형·거래형 중 하나로 분류하고, 거기에 맞는 글 유형 템플릿을 고른다
  - 상위 글들이 공통으로 다루는 항목(빠지면 안 되는 것)과 아무도 안 다루는 항목(차별점)을 나눠 적는다
  - 같은 키워드로 이미 노출 중인 내 글이 있으면 새 글을 쓰지 않고 `/refresh-post`로 돌린다 (자기 잠식 방지)
- 산출물: `00_brief.md`
- 통과 조건: 의도·유형·차별점 각 1개 이상 명시, 정책 재확인 통과

#### ② 리서치

- 담당: `researcher`
- 도구: WebSearch, WebFetch
- 전문성 장치 — 출처 등급 (`source-tiers.md`)
  - 1등급: 법령, 정부·공공기관, 공식 공고, 제조사·서비스 공식 문서 → 수치·조건·기간의 유일한 근거
  - 2등급: 주요 언론, 협회·연구기관 → 배경 설명
  - 3등급: 개인 블로그, 커뮤니티 → 아이디어 참고만, 사실 근거로 사용 금지
  - 수치·금액·기간은 1등급 출처 하나 이상, 가능하면 둘로 교차 확인
  - 출처 문장을 그대로 옮겨 적지 않고 요지와 값만 기록 (복제 방지)
- 산출물: `01_research.md` (사실 ID, 내용, 값, 출처 URL, 등급, 확인일)
- 통과 조건: 브리프의 핵심 질문마다 근거 사실 1개 이상

#### ③ 경험·자료 입력 (사람)

- 에이전트가 질문 3개와 필요한 사진·스크린샷 목록을 만든다. 예: "직접 신청해 봤다면 걸린 시간은?", "헷갈렸던 단계는?", "신청 완료 화면 캡처(개인정보 가림)"
- 답은 `02_experience.md`, 사진은 `images/raw/`에 넣는다
- 비어 있으면 "경험 없음"으로 적고 정보형으로만 쓴다. 이 경우 검토 대기함에 "경험 보강 권장" 표시

#### ④ SEO 아웃라인

- 담당: `writer` + `seo-structure`
- 전문성 장치: 6.3의 콘텐츠 구조를 그대로 적용. 첫 문단 즉답, H2 3~7개, 표 위치, FAQ 질문 후보(실제 검색어·연관 질문 기반), 이미지 위치 표시
- 산출물: `03_outline.md` (제목 후보 3개, 헤딩 트리, 섹션별 근거 사실 ID 매핑)
- 통과 조건: 모든 섹션에 근거 사실 ID 또는 경험 항목이 연결됨

#### ⑤ 초안

- 담당: `writer`
- 전문성 장치: `style-guide.md`(본인 글 3편에서 추출한 말투) + im-not-ai `quick-rules.md`를 쓰는 시점부터 지킨다. 이렇게 하면 ⑧이 대부분 light(1콜) 경로로 끝난다
- 산출물: `04_draft.md`

#### ⑥ 사실 검증

- 담당: `fact-checker`
- 전문성 장치: 초안의 사실 주장을 문장 단위로 뽑아 `01_research.md`와 대조. 수치·날짜·금액·조건·고유명사는 전수 확인. 근거 없는 문장은 삭제하거나 "확인 필요"로 표시. 건강·금융·법률이면 단정 표현을 조건 표현으로 바꾼다
- 산출물: `05_factcheck.md` (주장 / 근거 ID / 판정), 수정된 초안
- 통과 조건: 불일치 0건

#### ⑦ 정책 검수 (본문 기준)

- 담당: `policy-reviewer` + `adsense-policy-check`
- 전문성 장치: 주제가 괜찮아도 본문 서술이 문제가 될 수 있다. 예: 절약 글에서 "공유 계정으로 OTT 싸게 보기", 건강 글에서 "이걸 먹으면 낫는다". 6.1 사전으로 1차 키워드 검사 후 에이전트가 맥락 판정
- 산출물: `06_policy.md`
- 통과 조건: BLOCK·AVOID 0건. CAUTION이면 필수 장치(기준일, 공식 출처, 면책 문구) 확인

#### ⑧ 윤문 — im-not-ai

4.4 참고.

#### ⑨ 이미지 기획·제작

- 담당: `visual-designer` + `image-studio`
- 도구: Playwright(HTML 템플릿 캡처), Pillow(합성·주석·WebP 변환), matplotlib(차트), Gemini API(일러스트), Figma MCP(선택, 썸네일 템플릿 시안)
- 상세는 4.5

#### ⑩ SEO 마크업·HTML 변환

- 담당: `seo-editor` + `tistory-format`
- 도구: `md_to_tistory.py`, `internal_links.py`(`published.json` + `gsc.sqlite`로 관련 글 선정)
- 전문성 장치: 6.3 마크업 규칙 적용. 목차 앵커, 표의 `thead`·`th`, 이미지 자리표시, 외부 출처 링크, 관련 글 2~3개
- 산출물: `post.html`, `meta.json` (제목, 태그 10개 이내, 카테고리, 대표 이미지, 이미지별 alt·캡션), `preview.html`

#### ⑪ QA 게이트

4.6 체크리스트를 `qa_check.py`가 코드로 검사. 실패하면 해당 단계로 되돌려 최대 2회 재시도, 그래도 실패하면 사람 검토로 넘긴다.

#### ⑫ 검토·발행 (사람, A방식)

7장 참고.

#### ⑬ 사후 처리

- `update_published.py`로 발행 URL 기록
- `gsc_inspect.py`가 D+3, D+7, D+14에 URL Inspection API로 색인 상태 확인
- 14일이 지나도 미색인이면 주간 리포트에 올려 내부 링크 추가·품질 점검

### 4.4 im-not-ai 연결 (⑧)

- 설치: 프로젝트 `.claude/settings.json`에 마켓플레이스와 플러그인을 등록해 로컬·클라우드 어디서 열어도 같은 버전이 깔리게 한다. 수동 설치라면 `/plugin marketplace add epoko77-ai/im-not-ai` → `/plugin install humanize-korean@im-not-ai`
- 호출: `/humanize-korean 장르: 블로그` + 검증된 초안. `_workspace/{run_id}/final.md`를 `07_humanized.md`로 복사
- 경로 선택은 스킬의 `route_hint`에 맡긴다 (light 1콜 / standard 2콜 / heavy 3콜 이상)
- 게이트 처리
  - exit 0 → 진행
  - exit 1 → finalize 결과까지 보고 진행, 경고는 검토 대기함 메모에 남김
  - exit 2(변경률 50% 이상) → 윤문본을 버리고 검증된 초안으로 1회 재시도, 또 실패하면 사람 검토
  - 등급 C/D → `--strict`(heavy)로 1회 재실행, 그래도 C/D면 사람 검토
- 충돌 지점: im-not-ai는 과한 헤딩·불릿·볼드를 AI 티로 본다. 블로그 SEO에는 H2/H3와 표가 필요하다. 윤문 지시에 "헤딩과 표 구조는 유지"를 넣고, 헤딩 때문에 변경률이 부풀면 `--ignore-markup` 수치도 기록한다. `qa_check.py`가 윤문 전후 H2 목록이 같은지 확인한다
- 윤문 뒤 사실 재대조: im-not-ai 게이트가 수치 보존을 보지만, `qa_check.py`에서 `01_research.md` 값과 한 번 더 대조한다
- 버전 고정: 동작 확인한 버전(현재 2.3.2)을 `blog.yaml`에 적고, 올릴 때는 샘플 글 3편으로 회귀 확인

### 4.5 이미지 제작 (⑨)

#### 썸네일 (글당 1장, 필수)

- 방식: 카테고리별 HTML/CSS 템플릿 2~3종 → Playwright로 PNG 캡처 → WebP 변환
- 이미지 생성 AI가 아니라 템플릿을 쓰는 이유: 한글이 깨지지 않고, 목록에서 블로그가 한눈에 알아보이며, 비용이 들지 않고, 같은 입력이면 같은 결과가 나온다
- 템플릿 시안은 처음에 한 번만 만든다. Figma MCP로 디자인한 뒤 HTML로 옮기거나, 바로 HTML로 만든다
- 구성: 메인 문구(2줄, 줄당 12자 안팎), 보조 문구, 카테고리 라벨, 블로그 표기. 문구는 제목을 그대로 쓰지 않고 핵심 이득을 짧게 요약
- 규격: 1200×675(16:9) 원본. 스킨 목록이 정사각형으로 자르는 경우를 대비해 가운데 675×675 안에 핵심 문구 배치 (0주차에 스킨 썸네일 비율 확인)
- 폰트: 상업 이용이 가능한 OFL 폰트만 (예: Pretendard, Noto Sans KR). 폰트 저작권 분쟁이 국내 블로그에서 자주 생기는 문제라 `image-guidelines.md`에 허용 목록을 둔다

#### 본문 이미지 (글당 2~5장)

| 유형 | 언제 | 제작 방법 | 규칙 |
|---|---|---|---|
| 요약 카드 | 거의 모든 글 (도입 또는 정리) | HTML 템플릿 → Playwright | 카드 속 내용은 본문 텍스트에도 반드시 있음 |
| 단계 흐름도 | 방법형 | HTML/SVG 템플릿 → Playwright | 단계 수·명칭이 본문 H3와 일치 |
| 비교 카드 | 비교형 | HTML 표 스타일 캡처 | 본문에 HTML `<table>`을 반드시 함께 둔다 (이미지 속 글자는 검색엔진이 잘 못 읽음) |
| 데이터 차트 | 수치가 있는 글 | matplotlib 스크립트 | `01_research.md` 수치만 사용, 출처·기준일을 차트 아래 표기 |
| 개념 일러스트 | 도입부, 글당 0~1장 | Gemini API (`gen_illustration.py`). 수동 대안은 Gemini 앱에서 만들어 `images/raw/`에 넣기 | 글자·실존 인물·로고·브랜드 금지. 사실 정보를 그림으로 대신하지 않음 |
| 실제 사진·스크린샷 | 경험형·방법형 | 사람이 제공 → `annotate.py`(화살표·박스·가림 처리) | 주민번호·계좌·얼굴·차량번호 등 개인정보 가림. 경험의 증거라 SEO 가치가 가장 큼 |

#### 공통 후처리 (`optimize_images.py`)

- WebP 변환, 가로 1200px 이하, 장당 200KB 안팎 목표
- 파일명은 내용을 설명하는 영문 슬러그 (예: `year-end-tax-02-steps.webp`)
- alt 텍스트: 이미지가 보여주는 내용을 한 문장으로. 키워드 나열 금지
- 개인 사진의 EXIF 위치정보 제거
- AI 생성 이미지는 IPTC `DigitalSourceType`을 `trainedAlgorithmicMedia`로 기록 (구글이 권장하는 AI 생성 표시 방식)
- 스톡 이미지를 쓰면 출처·라이선스를 `images/credits.json`에 기록
- 선정적이거나 충격적인 이미지는 정책상 금지 (애드센스 제한사항은 이미지에도 적용)

이미지 기획서(`08_image_plan.json`) 예시

```json
[
  {"id": "thumb", "type": "thumbnail", "template": "guide-blue",
   "headline": ["연말정산 간소화", "5분 만에 끝내기"], "label": "세금",
   "alt": "연말정산 간소화 서비스 이용 방법 썸네일"},
  {"id": "img2", "type": "steps", "after_heading": "신청 방법",
   "steps": ["홈택스 로그인", "간소화 자료 조회", "공제 신고서 작성", "회사 제출"],
   "alt": "연말정산 간소화 서비스 4단계 흐름도", "caption": "신청은 4단계로 끝난다"}
]
```

### 4.6 QA 체크리스트 (`qa_check.py`)

본문
- 길이가 유형별 범위 안 (`blog.yaml`에서 조정)
- H2 3~7개, 헤딩 단계 건너뛰기 없음(H2 다음 바로 H4 금지), 윤문 전후 H2 목록 동일
- 첫 문단이 검색 의도에 즉답하고, 120자 안팎에서 핵심이 끝남
- 숫자·날짜·금액이 `01_research.md` 값과 일치
- 출처 원문과 15어절 이상 연속 일치하는 문장 없음
- 챗봇 잔재 문구 없음 ("도움이 되셨길", "AI로서", "물론입니다" 등)

정책
- `adsense-topic-policy.yaml` 키워드 사전 검사 통과 (본문·제목·alt·태그 전부)
- 클릭 유도 문구 없음 ("광고 클릭", "아래 배너를 눌러" 등)
- CAUTION 주제면 기준일·공식 출처·면책 문구 존재
- 제휴·추적 링크 없음 (제휴는 현재 하지 않음)

마크업·이미지
- 남은 마크다운 기호, 깨진 HTML, 빈 링크 없음
- 목차 앵커와 H2 `id`가 서로 맞음
- 표에 `thead`와 `th` 존재
- 이미지마다 alt·파일 크기·라이선스 기록 확인, 썸네일 존재
- 내부 링크 2개 이상, 링크 텍스트가 "여기", "클릭"이 아님

### 4.7 외부 API 준비 (0주차)

Search Console API 설정은 9장에 있다. 나머지 세 가지는 아래와 같다. 키 값은 모두 저장소 밖(로컬 환경 변수 또는 클라우드 환경 변수)에 둔다.

#### 네이버 검색광고 API — 키워드 도구 (계정 있음)

1. 네이버 검색광고 시스템 → 도구 → API 사용 관리에서 API 라이선스를 발급한다
2. 고객 ID(CUSTOMER_ID), 액세스 라이선스(API 키), 비밀키(SECRET_KEY) 세 값을 환경 변수로 등록한다
3. `naver_keywords.py`가 키워드 도구(`/keywordstool`)를 호출한다. 요청마다 타임스탬프·메서드·경로를 비밀키로 HMAC-SHA256 서명해 헤더에 넣는다
4. 받는 값: 연관 키워드, PC·모바일 월간 검색수, 월평균 클릭수·클릭률, 광고 노출 깊이, 경쟁 정도
5. 쓰는 법
   - 월간 검색수 → 수요
   - 경쟁 정도·광고 노출 깊이 → 광고주가 몰리는 정도. 4.3 ⓪ 점수식의 "단가 등급"을 처음에는 이 값으로 대신하고, 3개월차부터 자체 RPM으로 보정한다
   - 연관 키워드 → 긴 키워드 후보 발굴
6. 주의: 한 번에 넣는 힌트 키워드 수에 제한이 있고, 검색수가 아주 적은 키워드는 숫자 대신 "< 10" 같은 문자열로 온다. 스크립트가 둘 다 처리한다
7. 한계: 네이버 사용자 기준 데이터다. 티스토리 유입은 구글·다음 비중이 크므로, 실제 구글 성과는 Search Console로 본다

#### 네이버 데이터랩 검색어 트렌드 API

- 검색광고와 별개다. 네이버 개발자센터에서 애플리케이션을 등록하고 Client ID·Client Secret을 받는다
- `naver_trend.py`: 키워드 묶음의 기간별 상대 검색량 → 시즌이 시작되는 시점을 잡아 발행 시기를 정한다

#### Gemini API — 이미지 생성 (Google AI Pro 구독 활용)

- Gemini 앱 구독(Google AI Pro)에는 API 키가 들어 있지 않다. 대신 2026년 1월부터 Google AI Pro에 Google Developer Program 프리미엄 혜택이 포함돼, 매달 Google Cloud 크레딧 $10이 나오고 이걸 Gemini API(AI Studio·Vertex AI)에 쓸 수 있다
- 절차
  1. Google Developer Program의 혜택 페이지(`developers.google.com/program/my-benefits`)에서 혜택 활성화
  2. Google Cloud 결제 계정 연결 (Search Console API와 같은 프로젝트를 써도 된다)
  3. AI Studio에서 API 키 발급, 크레딧이 그 프로젝트에 적용되는지 확인
  4. 결제 예산 알림을 $10에 걸어, 크레딧을 넘기면 바로 알 수 있게 한다
- 비용 감: 개념 일러스트는 글당 0~1장, 한 달 20장 안팎이다. 제3자 정리 기준 장당 약 $0.04~0.07이라 월 크레딧 안에 들어올 가능성이 크다. 최신 가격은 공식 가격표로 확인한다
- 주의: 최신 이미지 모델은 API 무료 등급이 없다는 정리가 많다. 결제 계정 없이 쓸 수 있다고 가정하지 않는다. 결제 계정 조건(선불 충전 등)은 바뀔 수 있어 설정 화면에서 확인한다
- `gen_illustration.py`: 모델명·해상도는 `blog.yaml`에 두고, 생성 이미지에는 IPTC AI 생성 표시를 넣고 프롬프트를 `images/credits.json`에 남긴다
- 수동 대안: Gemini 앱에서 직접 만들어 `images/raw/`에 넣어도 된다. 자동화는 안 되지만 구독 범위 안이라 추가 비용이 없다

---

## 5. 리포지토리 구조

공개 저장소인 `codingStudy.github.io` 대신 **새 private GitHub 저장소**(예: `lkj1017/tistory-agent`)에 둔다. 초안, 검색 성과 데이터, API 키 경로가 들어가기 때문이다. 1단계(로컬)에서는 버전 관리와 백업을 맡고, 2단계(클라우드 예약 생성)에서는 클라우드 세션이 이 저장소를 받아 작업하고 글마다 PR을 올린다.

```
tistory-agent/
├─ CLAUDE.md                        # 블로그 정체성, 말투, 금지 사항, 파이프라인 규칙
├─ .claude/
│  ├─ settings.json                 # im-not-ai 플러그인 등록, 권한
│  ├─ skills/
│  │  ├─ plan-week/SKILL.md
│  │  ├─ new-post/SKILL.md
│  │  ├─ adsense-policy-check/SKILL.md
│  │  ├─ seo-structure/SKILL.md
│  │  ├─ image-studio/SKILL.md
│  │  ├─ tistory-format/SKILL.md
│  │  ├─ gsc-insights/SKILL.md
│  │  ├─ refresh-post/SKILL.md
│  │  └─ weekly-report/SKILL.md
│  └─ agents/
│     ├─ topic-strategist.md  serp-analyst.md  researcher.md  writer.md
│     └─ fact-checker.md  policy-reviewer.md  visual-designer.md  seo-editor.md
├─ config/
│  ├─ blog.yaml                     # 카테고리, 발행 빈도, 길이 범위, im-not-ai 버전
│  ├─ style-guide.md                # 본인 글 3편에서 추출한 말투
│  ├─ source-tiers.md               # 출처 등급 규칙
│  ├─ policy/
│  │  ├─ adsense-topic-policy.yaml  # BLOCK/AVOID/CAUTION 범주와 한국어 키워드 사전
│  │  └─ policy-notes.md            # 공식 정책 요약 + 원문 링크 + 마지막 확인일
│  ├─ seo/
│  │  ├─ markup-rules.md            # 6.3 마크업 규칙
│  │  └─ templates/                 # howto, guide, compare, review, checklist 아웃라인
│  └─ images/
│     ├─ image-guidelines.md        # 유형별 규칙, 허용 폰트 목록
│     └─ templates/                 # 썸네일·요약 카드·흐름도·비교 카드 HTML/CSS
├─ data/
│  ├─ backlog.csv                   # 키워드, 의도, 유형, 시즌, 점수, 정책 판정, 상태
│  ├─ published.json                # 발행 글 URL·제목·키워드·발행일
│  └─ gsc.sqlite                    # Search Console 일별 데이터 (16개월 보관 한도를 넘어 누적)
├─ posts/
│  └─ 2026-11-03-year-end-tax/
│     ├─ 00_brief.md  01_research.md  02_experience.md  03_outline.md  04_draft.md
│     ├─ 05_factcheck.md  06_policy.md  07_humanized.md  08_image_plan.json
│     ├─ post.html  meta.json  preview.html  qa.json
│     └─ images/ (raw/, 최종 webp, credits.json)
├─ scripts/
│  ├─ gsc_sync.py  gsc_query.py  gsc_inspect.py  gsc_sitemaps.py
│  ├─ naver_keywords.py  naver_trend.py
│  ├─ render_template.py  annotate.py  make_chart.py  gen_illustration.py  optimize_images.py
│  ├─ md_to_tistory.py  internal_links.py  build_preview.py
│  ├─ qa_check.py  update_published.py
│  └─ requirements.txt
├─ .env.example                     # 키 이름만 (GSC_SA_KEY_PATH, GSC_SITE_URL, NAVER_AD_CUSTOMER_ID,
│                                   #   NAVER_AD_ACCESS_LICENSE, NAVER_AD_SECRET_KEY, NAVER_DATALAB_CLIENT_ID,
│                                   #   NAVER_DATALAB_CLIENT_SECRET, GEMINI_API_KEY). 실제 값은 커밋하지 않음
└─ _workspace/                      # im-not-ai 산출물 (.gitignore)
```

### CLAUDE.md 핵심 규칙 (초안)

```markdown
# 목적
lifegoals001.com 티스토리 글을 만든다. 사람이 검토하기 전에는 어떤 글도 발행하지 않는다.

# 절대 규칙
- config/policy/adsense-topic-policy.yaml의 BLOCK·AVOID 범주는 주제든 본문 일부든 다루지 않는다.
- 수치·날짜·금액·조건은 1등급 출처가 있는 것만 쓴다. 출처 문장을 그대로 옮기지 않는다.
- 02_experience.md에 없는 경험을 지어내지 않는다.
- 건강·금융·법률 주제는 기준일과 공식 출처를 적고, 단정 대신 조건을 쓴다.

# 파이프라인
- /new-post 단계를 건너뛰지 않는다. 사실 검증(⑥)과 정책 검수(⑦) 전에 윤문(⑧)하지 않는다.
- 윤문은 /humanize-korean 장르: 블로그, 헤딩·표 구조 유지. 게이트 결과는 qa.json에 기록한다.
- 계산·검사·렌더링은 scripts/를 쓰고, 결과를 에이전트가 추정하지 않는다.
- 하루 발행 예정 글은 1편, 많아도 2편.
```

---

## 6. 콘텐츠 전략

### 6.1 애드센스 주제 정책 — 하드 필터

이 블로그의 수익은 애드센스 계정 하나에 달려 있다. 그래서 정책 위험이 있는 주제는 트래픽이 아무리 커도 다루지 않는다. 판정은 네 단계다.

| 판정 | 의미 | 처리 |
|---|---|---|
| BLOCK | Google 게시자 정책(금지 콘텐츠) 위반. 광고 게재 중지, 반복되면 계정 해지 위험 | 무조건 탈락 |
| AVOID | Google 게시자 제한사항. 정책 위반은 아니지만 광고가 제한돼 수익이 거의 안 나고 사이트 평판에도 불리 | 무조건 탈락 |
| CAUTION | 허용되지만 오류 시 피해가 크고 정책 해석 여지가 있는 주제 (YMYL 등) | 필수 장치를 갖춘 경우만 진행 |
| ALLOW | 일반 주제 | 진행 |

#### BLOCK — 게시자 정책 금지 콘텐츠

| 범주 | 피해야 할 한국 블로그 주제 예시 |
|---|---|
| 불법 콘텐츠 | 사설 토토·불법 도박 사이트, 불법 스트리밍·웹툰 사이트 주소, 토렌트, 마약 구입, 해킹·계정 탈취 방법, 불법 촬영물 |
| 지적재산권 침해 | 영화·드라마 무료 다시보기 링크, 기사·타 블로그 전문 복제, 소프트웨어 크랙·정품 인증 우회, 레플리카(모조품) 구매처 |
| 멸종위기종 | 상아·코뿔소 뿔 등 보호종 제품 판매 |
| 위험하거나 비방하는 콘텐츠 | 특정 집단 혐오·비하, 괴롭힘·신상 털기, 자해·자살 방법, 폭력 선동, 동물 학대 |
| 부정행위 조장 | 대리 시험·대리 과제, 가짜 진단서·위조 서류, 몰래 설치하는 위치 추적·스파이 앱, 리뷰 조작, 시험 문제 유출 |
| 허위 표시·신뢰할 수 없는 주장 | 근거 없는 질병 치료 주장(민간요법으로 암 치료 등), 백신·공중보건 허위 정보, 선거 허위 정보, 기후변화 과학적 합의 부정, 조작된 이미지·영상, 낚시성 제목, 정부 지원금을 사칭한 피싱성 안내 |
| 악성·원치 않는 소프트웨어 | 출처 불명 APK·크랙 배포, 키젠 |
| 성적 콘텐츠 관련 금지 항목 | 노골적 성적 콘텐츠, 조건만남·성매매, 국제결혼 중개(우편 주문 신부), 아동 대상 성적 콘텐츠 |

#### AVOID — 게시자 제한사항

성적인 콘텐츠(선정적 표현), 충격적인 콘텐츠(사고·시신·잔혹 이미지), 폭발물, 총기·총기 부품, 기타 무기(전기충격기·호신용 스프레이 판매 등), 담배(전자담배 포함), 기분 전환용 약물(대마 등), 주류 판매·오용, 온라인 도박(합법 카지노·스포츠 베팅 포함), 처방 의약품(전문의약품 구매·해외 직구), 미승인 의약품·보조제(해외 다이어트약 등).

#### CAUTION — 진행 가능, 필수 장치 있음

| 주제 | 필수 장치 |
|---|---|
| 건강·의료 정보 | 공공기관·학회 출처만, 치료 효과 단정 금지, "진료를 대신하지 않음" 문구 |
| 금융·투자·코인 | 종목·코인 추천과 수익 보장 표현 금지, 금리·조건 기준일, 투자 권유 아님 문구 |
| 대출 | 정부·금융기관 공식 상품 정보만, 대출 중개·불법 사금융 연결 금지 |
| 법률·세금 | 법령·국세청 등 1등급 출처, 기준 연도, 개별 상담 대체 아님 문구 |
| 사건·사고·재난 | 비극을 트래픽에 이용하지 않음, 자극적 이미지 금지, 광고주 기피가 커서 되도록 피함 |
| 정치·선거 | 선거 기간에는 다루지 않음 |

#### 계정을 지키는 행동 규칙 (주제와 별개)

- 본인 광고 클릭 금지, 지인에게 클릭 부탁 금지
- 트래픽 교환·자동 방문·유료 트래픽 구매 금지
- 광고 근처에 "클릭", "다운로드" 같은 버튼형 요소나 오해를 부르는 이미지 배치 금지
- 저작권 신고를 받으면 즉시 확인하고 내린다

#### 구현

- `adsense-topic-policy.yaml`: 범주별 정의, 한국어 키워드 사전(예: "토토", "다시보기", "크랙", "직구 약", "대리 과제"), 판정 예시
- 검사 지점 4곳: ⓪ 주제 후보, ① 브리프, ⑦ 본문, ⑪ 최종 HTML(제목·alt·태그 포함)
- 키워드 사전은 1차 거름망이고, 맥락 판정은 `policy-reviewer`가 한다. "도박 중독 상담 기관 안내"처럼 키워드는 걸리지만 괜찮을 수 있는 경우도, 애매하면 탈락시킨다
- 정책 원문은 분기마다 다시 확인해 `policy-notes.md`의 마지막 확인일을 갱신한다

위 범주는 Google 고객센터의 게시자 정책·게시자 제한사항 목차를 기준으로 정리했다. 이번 작업 환경에서는 원문 페이지가 네트워크 정책에 막혀 직접 열지 못했으므로, 0주차에 원문과 한 번 대조한다.

### 6.2 주력 카테고리 (확정)

블로그 화면을 보지 못했고 검색 색인에도 글이 거의 잡히지 않아서, 새로 키우는 블로그로 보고 골랐다. 기준은 다섯 가지다: 정책 안전성, 광고 단가, 검색 수요가 해마다 반복되는지, 이 파이프라인(1등급 출처 리서치·사실 검증)과 맞는지, 도메인 이름(lifegoals)과 이어지는지.

| | 주력 A: 생활 돈 관리·제도 활용 | 주력 B: 목표 달성·자기계발 도구 |
|---|---|---|
| 비중 | 60% | 40% |
| 역할 | 트래픽과 수익의 중심 | 블로그 정체성, 경험 기반 신뢰, 경쟁이 약한 틈새 |
| 다루는 것 | 정부·지자체 지원 제도 신청법, 생활비 절감 제도(교통비·통신비·에너지), 생활 세금(연말정산, 종합소득세 환급, 자동차세 연납) | 목표 설정·습관·시간 관리 방법, 생산성 앱·노션·캘린더 활용법, 자격증 시험 일정과 공부 계획 |
| 광고 단가 | 중~높음 (금융·통신 광고주가 붙는 맥락) | 중간 (교육·소프트웨어 광고) |
| 정책 판정 | 대부분 ALLOW, 세금은 CAUTION | ALLOW |
| 파이프라인과의 궁합 | 공식 공고·법령을 정리하는 일이라 리서치·사실 검증이 그대로 품질이 된다 | 직접 써 본 경험이 핵심이라 ③ 경험 입력이 차별점이 된다 |
| 약점과 대응 | 큰 키워드는 경쟁이 세다 → 조건·상황이 붙은 긴 키워드로 공략하고, 공고가 나오면 빨리 발행 | 검색량이 상대적으로 적다 → 연말·연초 시즌에 집중 |

두 주력을 잇는 고리는 "돈 목표 세우기"(예산 짜기, 비상금 목표, 연간 절약 계획)다. 양쪽 허브에서 서로 링크한다.

주력 안이라도 다루지 않는 것: 대출 상품 비교·추천, 주식·코인 종목, 보험 상품 추천, 건강·다이어트, 연예·사건 이슈. 정책상 CAUTION 이상이거나, 틀렸을 때 피해가 크고 새 블로그가 신뢰를 얻기 어려운 영역이다.

#### 주제 허브와 첫 주제 후보

허브마다 기둥 글 1편(총정리)과 세부 글 여러 편을 내부 링크로 묶는다. 주제 묶음이 쌓여야 블로그 전체의 전문성이 드러난다. 아래는 0주차에 네이버 키워드 도구로 검색량·경쟁도를 확인할 첫 후보이고, 확정 목록은 아니다.

| 허브 | 기둥 글 | 세부 글 후보 |
|---|---|---|
| A1 생활 세금 | 연말정산 총정리 (해마다 갱신) | 연말정산 미리보기 쓰는 법, 간소화 자료에 안 나오는 공제 항목, 중도 퇴사자 연말정산, 프리랜서 종합소득세 환급 조회, 자동차세 연납 신청법 |
| A2 생활비 절감 제도 | 교통비·통신비·에너지 지원 제도 한눈에 | 대중교통비 환급 제도 신청과 환급 확인, 알뜰폰 요금제 고르는 기준, 에너지바우처 대상·신청, 문화누리카드 사용처 |
| A3 지원 제도 찾기 | 내가 받을 수 있는 지원 제도 찾는 법 | 복지로·정부24에서 지원금 조회하기, 청년 대상 제도별 신청 방법, 지자체 지원금 찾는 법 |
| B1 목표·습관 | 목표 세우는 법 총정리 | 새해 목표 세우기 템플릿, 습관 추적 방법, 월간 회고 하는 법 |
| B2 생산성 도구 | 목표 관리 앱·도구 비교 | 노션 목표 관리 템플릿 만들기, 구글 캘린더 시간 블록 쓰는 법, 할 일 앱 비교 |
| B3 자격증·공부 | 2027 자격증 시험 일정 정리 | 시험 접수 방법, 직장인 공부 계획 짜기, 국비 지원 교육 찾는 법 |

#### 시즌 캘린더 (지금이 10월 말이라 바로 쓸 수 있는 것부터)

| 시기 | 발행할 주제 | 이유 |
|---|---|---|
| 11월 | 연말정산 미리보기·절세 준비, 에너지바우처, 새해 다이어리·플래너 도구 | 연말정산 미리보기 서비스가 보통 11월에 열리고, 겨울철 수요가 붙음 |
| 12월 | 새해 목표 세우기, 내년에 달라지는 생활 제도, 2027 자격증 시험 일정, 문화누리카드 잔액 사용 | 연말·연초 검색 급증 |
| 1월 | 연말정산 간소화·공제 항목, 자동차세 연납, 습관 만들기 | 간소화 서비스가 보통 1월 중순에 열림 |
| 2~3월 | 연말정산 환급·추가 공제, 봄철 지원 제도 공고 | 환급 시기, 새 공고 |
| 5월 | 종합소득세 신고·환급 | 신고 기간 |

정확한 날짜는 해마다 공식 공고로 확인하고, 검색량이 오르기 2~4주 전에 발행한다.

#### 운영 원칙

- 꾸준히 검색되는 글 70%, 시즌 글 30%
- 0주차: 위 후보를 포함해 30개 안팎을 네이버 키워드 도구로 돌려 검색량·경쟁 정도를 받고, Search Console에 이미 노출되는 쿼리와 합쳐 첫 4주 backlog를 확정한다
- 3개월차: 카테고리별 RPM과 Search Console 성과를 보고 A:B 비중을 다시 정한다
- 기존 글이 다른 분야라면, 주력과 이어지는 글은 허브에 연결하고 나머지는 그대로 둔다

### 6.3 구글 권장 SEO 구조

구글 Search Central 문서(SEO 기본 가이드, 유용하고 신뢰할 수 있는 사람 중심 콘텐츠, 이미지 SEO, 구조화 데이터)의 권고를 글 구조와 마크업 규칙으로 옮겼다. 구글은 "특정 글자 수"나 "키워드 밀도" 같은 공식 기준을 두지 않는다. 핵심은 검색한 사람이 다른 곳을 더 찾아볼 필요가 없게 만드는 것이다.

#### 콘텐츠 원칙

1. 사람 중심: 검색 의도에 첫 문단에서 답하고, 이어서 근거와 세부를 준다
2. 독창성: 상위 글에 없는 정보(직접 해 본 결과, 계산 예시, 정리한 표)를 최소 1개 넣는다
3. 경험·전문성·신뢰(E-E-A-T): 직접 찍은 사진, 작성자 소개, 1등급 출처 링크, 작성일·업데이트일 표시
4. 완결성: 상위 글들이 공통으로 다루는 항목은 빠짐없이 다룬다 (①의 공통 항목 목록)
5. 최신성: 기준일을 적고, 제도가 바뀌면 `/refresh-post`로 갱신. 날짜만 바꾸는 가짜 업데이트는 하지 않는다
6. 투명성: AI 도움을 받았는지 독자가 궁금해할 만한 글이면, 작성 방식을 짧게 밝히는 것을 고려한다 (구글이 권하는 Who·How·Why 관점)

#### 글 구조 (모든 유형 공통)

```
제목 (H1, 스킨이 출력)
├─ 첫 문단: 검색 의도에 즉답 (120자 안팎, 설명 스니펫 후보)
├─ 요약 박스: 핵심 3~5줄 + 기준일·출처
├─ 대표 이미지 (요약 카드 또는 썸네일과 같은 계열)
├─ 목차 (H2 앵커 링크)
├─ H2 본문 섹션 3~7개
│   ├─ H3 하위 항목
│   ├─ 표 (비교·조건·기간)
│   └─ 본문 이미지 (관련 문단 바로 옆)
├─ H2 자주 묻는 질문 (실제 검색어 기반 3~5개)
├─ H2 정리 (다음 행동 한 가지 제시)
├─ 참고 자료 (1등급 출처 링크)
├─ 작성자·업데이트 정보
└─ 관련 글 2~3개 (내부 링크)
```

#### 마크업 규칙 (`config/seo/markup-rules.md`)

| 요소 | 규칙 | 근거 |
|---|---|---|
| 제목 | 글마다 고유, 내용을 정확히 설명, 핵심 키워드는 자연스럽게 앞쪽에, 키워드 반복 금지. 모바일 검색 결과에서 잘리지 않게 30자 안팎 | 제목이 내용과 어긋나면 구글이 검색 결과 제목을 바꿔 표시 |
| 설명 | 티스토리는 본문 앞부분으로 설명을 만드는 경우가 많으므로 첫 문단을 설명문처럼 작성 (0주차 확인) | 구글은 페이지 내용이나 meta description으로 스니펫 생성 |
| 헤딩 | 본문은 H2부터 시작, H2 → H3 순서 지키기, 헤딩은 섹션 내용을 그대로 설명하는 문장 | 구조 파악과 접근성 |
| 목차 | H2마다 `id`, 목차는 `<a href="#id">` | 검색 결과의 섹션 바로가기 링크에 도움 |
| 표 | `<table>` + `<caption>` + `<thead>`의 `<th scope="col">`. 표를 이미지로만 넣지 않기 | 표 데이터는 텍스트여야 색인됨 |
| 이미지 | `<img>` + 설명형 alt + `width`·`height`, 첫 이미지는 지연 로딩 제외, 관련 문단 근처 배치, `<figure>`·`<figcaption>` | 구글 이미지 SEO 권고, 레이아웃 흔들림(CLS) 방지 |
| 링크 | 내부 링크는 설명형 앵커 텍스트, 외부 출처는 공식 사이트로. 제휴 링크는 쓰지 않음 (나중에 도입하면 `rel="sponsored"`와 대가성 문구 필수) | 링크 텍스트 권고, 유료 링크 표시 의무 |
| 구조화 데이터 | `BlogPosting` JSON-LD (headline, image, datePublished, dateModified, author). 글마다 넣기보다 스킨 단위로 넣는 편이 관리가 쉬움. 스킨이 이미 출력하는지 먼저 확인 | 구글은 JSON-LD 권장 |
| FAQ 마크업 | FAQ는 본문에 두되 `FAQPage` 마크업은 선택. 일반 사이트는 리치 결과가 나오지 않음 | 2023년 8월 변경 |
| 날짜 | 본문에 "작성일·최종 업데이트" 표시, 실질적 수정일 때만 업데이트일 변경 | 구글은 페이지에 보이는 날짜도 참고 |

#### 본문 HTML 뼈대 (`md_to_tistory.py` 출력 형태)

```html
<p class="lg-lead">연말정산 간소화 자료는 1월 15일부터 홈택스에서 조회할 수 있고, 회사 제출까지 4단계면 끝납니다.</p>

<div class="lg-summary">
  <p><strong>핵심 정리</strong></p>
  <ul>
    <li>조회 시작: 2027년 1월 15일 (국세청 공지 기준)</li>
    <li>필요한 것: 공동인증서 또는 간편인증</li>
  </ul>
  <p class="lg-meta">기준일 2026-12-20 · 출처 <a href="https://www.nts.go.kr/">국세청</a></p>
</div>

<p>[이미지 1: year-end-tax-01-summary.webp]</p>

<div class="lg-toc">
  <p><strong>목차</strong></p>
  <ol>
    <li><a href="#s1">간소화 서비스로 조회되는 항목</a></li>
    <li><a href="#s2">신청 방법 4단계</a></li>
  </ol>
</div>

<h2 id="s1">간소화 서비스로 조회되는 항목</h2>
<p>...</p>
<table>
  <caption>간소화 자료 조회 항목</caption>
  <thead><tr><th scope="col">항목</th><th scope="col">자동 조회</th><th scope="col">직접 제출</th></tr></thead>
  <tbody><tr><td>의료비</td><td>대부분</td><td>안경 구입비 등</td></tr></tbody>
</table>

<h2 id="s2">신청 방법 4단계</h2>
<h3>1단계. 홈택스 로그인</h3>
<p>...</p>
<p>[이미지 2: year-end-tax-02-steps.webp]</p>

<h2 id="faq">자주 묻는 질문</h2>
<h3>회사가 간소화 자료를 직접 받나요?</h3>
<p>...</p>

<h2 id="wrap">정리</h2>
<p>...</p>

<div class="lg-sources">
  <p>참고 자료: <a href="https://www.nts.go.kr/">국세청 연말정산 안내</a></p>
  <p>작성 lifegoals · 최종 업데이트 2026-12-20</p>
</div>
```

- 날짜·수치는 형식 예시이며, 실제 글에서는 `01_research.md`의 1등급 출처 값만 쓴다
- `lg-` 접두어 클래스의 스타일은 티스토리 스킨 CSS에 한 번 정의해 두고, 글에는 인라인 스타일을 넣지 않는다
- `[이미지 N: 파일명]`은 붙여넣은 뒤 그 자리에 이미지를 올리라는 자리표시다 (7장)

#### 블로그 단위 기술 점검 (0주차 1회, 이후 분기 1회)

- Search Console에 사이트맵 제출 상태 확인 (`gsc_sitemaps.py`)
- 모바일에서 읽기 좋은 가벼운 반응형 스킨, 대표 글 Core Web Vitals 점검
- 스킨의 H1 출력 방식, `BlogPosting` 구조화 데이터 출력 여부, robots 메타 확인
- 작성자 소개 페이지(누가, 어떤 경험으로 쓰는지) 만들고 글 하단에서 연결
- 글 주소 형식: 이미 발행한 글이 있으면 바꾸지 않는다 (기존 주소가 깨짐)

### 6.4 글 유형 템플릿 (`config/seo/templates/`)

1. 방법형: 즉답 → 준비물·조건 → 단계별 방법(H3 + 스크린샷) → 자주 막히는 곳 → FAQ
2. 총정리형: 즉답 → 대상·조건·기간 표 → 항목별 설명 → 공식 링크 → FAQ
3. 비교형: 누구에게 무엇이 맞는지 결론 → 비교표 → 항목별 차이 → 추천 대상
4. 경험형: 상황 → 직접 해 본 과정 → 결과·비용·시간 → 아쉬운 점 → 팁
5. 체크리스트·계산형: 체크리스트 → 계산 예시 2~3개 → 주의 사항

### 6.5 수익 보강

- 광고 배치: 티스토리 수익 메뉴에서 애드센스 자동광고 + 본문 상·하단으로 시작. 광고가 많으면 체류 시간과 Core Web Vitals가 나빠진다
- 리라이트: 9장 분석에서 나온 후보를 매주 2편 이상 보강. 새 글보다 효율이 좋은 경우가 많다
- 제휴(쿠팡 파트너스 등)는 하지 않는다. 수익원은 애드센스 하나로 두고, 트래픽이 자리 잡은 뒤 다시 검토한다

---

## 7. 발행 방식 — A방식 (붙여넣기)

### 글 한 편 발행 절차 (5~8분)

1. `preview.html`을 브라우저로 연다. 실제 모양 미리보기와 함께 [제목 복사] [본문 HTML 복사] [태그 복사] 버튼, 이미지별 alt·캡션 복사 버튼이 있다
2. 미리보기로 최종 읽기. 고칠 곳이 있으면 `/new-post` 세션에 말해서 고친다 (`post.html`을 직접 고치면 QA를 다시 돌린다)
3. 티스토리 글쓰기 → 에디터를 HTML 모드로 바꾸고 본문 HTML 붙여넣기 → 기본 모드로 돌아온다
4. `[이미지 N: 파일명]` 자리표시를 하나씩 지우고 그 자리에 `posts/.../images/`의 파일을 올린다. alt 텍스트를 넣고, 썸네일을 대표 이미지로 지정
5. 제목, 카테고리, 태그 입력 → 공개 설정 → 발행 시간 예약
6. 발행 URL을 `update_published.py`에 넣는다 (또는 다음 `/new-post` 실행 때 묻게 한다)

이미지를 외부 서버에 올려 링크하지 않고 티스토리에 직접 올리는 이유: 이미지 검색 유입이 내 블로그로 들어오고, 외부 링크가 끊길 위험이 없다.

### 0주차 호환성 점검 (비공개 테스트 글 1편)

뼈대 HTML을 비공개 글로 붙여넣어 아래를 확인하고, 결과를 `markup-rules.md`에 반영한다.

- 에디터가 유지하는 태그·속성: `id`, `class`, `table`의 `caption`·`thead`·`th scope`, `figure`·`figcaption`
- 에디터 "제목1·2·3"이 각각 어떤 태그(h2/h3/h4)로 저장되는지, 스킨이 글 제목을 H1으로 출력하는지
- 이미지 alt 입력 방법과 HTML에 저장되는 형태
- 설명(meta description)이 어떻게 만들어지는지
- 스킨이 구조화 데이터를 출력하는지 (리치 결과 테스트로 확인)

---

## 8. 실행 위치와 자동화 수준 — 추천안

결론: **로컬에서 시작하고, 품질이 안정되면 "생성은 클라우드 예약 작업, 발행은 PC"로 전환한다.**

| 단계 | 기간 | 어디서 | 무엇을 | 사람 몫 |
|---|---|---|---|---|
| 1단계 | 0~3주차 | 본인 PC의 Claude Code (데스크톱 앱 또는 터미널) | `/new-post`를 직접 실행하며 템플릿·프롬프트·스타일 가이드 다듬기. 실행할 때마다 `gsc_sync.py`가 증분 동기화 | 주제 고르기, 글마다 검토·발행, 주 1회 `/weekly-report` 실행 |
| 2단계 | 4주차~ (전환 조건 충족 시) | Claude Code 클라우드 예약 작업 + private GitHub 저장소 | 평일 아침 `/new-post` 1편 자동 생성 → 글마다 PR. 월요일 아침 GSC 주간 리포트 + 다음 주 주제 후보 PR | 휴대폰으로 PR 읽고 승인·수정 요청, PC에서 `git pull` 후 미리보기 열어 붙여넣기 발행 |

이렇게 추천하는 이유

- 처음 몇 주는 템플릿과 프롬프트를 고치고 결과를 바로 열어 보는 반복이 가장 많다. 로컬이 가장 빠르다
- 지금 클라우드 환경은 공공기관 사이트·네이버 API·티스토리 접근이 막혀 있다(1장 실측). 리서치 품질이 1등급 출처에 달려 있어서, 클라우드로 옮기려면 네트워크 설정부터 바꿔야 한다
- 발행은 A방식이라 어차피 PC 브라우저에서 한다. 그래서 생성 위치만 클라우드로 옮기면 된다
- 로컬 cron으로 예약하면 PC가 켜져 있어야 한다. 매일 도는 작업은 클라우드 예약 작업이 더 안정적이다
- 2단계에서도 사람 검토와 발행은 그대로 남긴다. 3단계(무인 발행)는 두지 않는다

1단계 → 2단계 전환 조건 (모두 충족)

- 연속 10편이 QA를 1회에 통과
- 사람이 고치는 시간이 글당 5분 이내
- 정책 경고 0건, 사실 오류 0건

2단계 준비물

- 클라우드 환경 네트워크 설정: 접근 수준을 넓히거나, 허용 도메인에 리서치 대상(예: `gov.kr`, `nts.go.kr`, `bokjiro.go.kr`, `law.go.kr`, `q-net.or.kr`)과 `openapi.naver.com`, `api.searchad.naver.com`을 추가. Search Console API와 Gemini API가 쓰는 `googleapis.com` 계열은 이미 통과한다. 방법은 [클라우드 환경 네트워크 문서](https://code.claude.com/docs/en/cloud-environments#network-access) 참고
- 비밀값: Search Console 서비스 계정 키, 네이버 검색광고·데이터랩 키, Gemini API 키를 클라우드 환경 변수로 등록 (저장소에 넣지 않음)
- 예약 실행 중에는 사람에게 질문할 수 없으므로, ③ 경험 입력은 `/plan-week` PR에서 미리 받는다. 경험이 없는 주제는 정보형으로만 쓰고 우선순위를 낮춘다

비용: im-not-ai 런타임 에이전트는 기본 `model: opus`이고 경로에 따라 1~3콜을 쓴다. 리서치부터 이미지까지 글당 사용량을 1단계에서 실측해, 구독 사용량 안에서 하루 1편이 가능한지 보고 에이전트별 모델을 조정한다. Gemini 이미지 생성은 글당 0~1장이라 Google AI Pro 월 크레딧 $10 안에서 쓰는 것을 목표로 한다 (4.7).

---

## 9. 성과 피드백 루프 — Search Console API

### 0주차 설정

1. Google Cloud 프로젝트를 만들고 Search Console API 사용 설정
2. 서비스 계정을 만들고 JSON 키 발급. 로컬은 `~/.config/tistory-agent/gsc-sa.json`(저장소 밖), 클라우드는 환경 변수
3. Search Console → 설정 → 사용자 및 권한에서 서비스 계정 이메일을 사용자로 추가 (사이트맵 제출까지 하려면 "전체" 권한)
4. 속성 형식 확인: 도메인 속성이면 `sc-domain:lifegoals001.com`, URL 접두어 속성이면 `https://lifegoals001.com/`
5. `gsc_sync.py --backfill`로 조회 가능한 최대 기간(16개월)을 한 번에 받아 `gsc.sqlite`에 저장

### 쓰는 API

| API | 용도 | 주기 |
|---|---|---|
| `searchanalytics.query` | 날짜·페이지·쿼리·기기별 노출·클릭·CTR·순위. 검색 유형은 웹·이미지·디스커버(디스커버는 쿼리 차원 없음) | 매일 증분. 최근 3일은 데이터가 확정되지 않으므로 매번 다시 받아 덮어씀 |
| `urlInspection.index.inspect` | 새 글 색인 여부, 구글이 고른 canonical, 마지막 크롤링 시각 | 발행 후 D+3·D+7·D+14 |
| `sitemaps.list` / `submit` | 사이트맵 상태 확인·제출 | 주 1회 확인 |

Indexing API는 채용 공고·라이브 방송 전용이라 블로그 글에는 쓰지 않는다.

### 데이터 저장 (`gsc.sqlite`)

- `daily_page_query(date, page, query, device, clicks, impressions, ctr, position)`
- `daily_page(date, page, search_type, clicks, impressions, ctr, position)` — 쿼리 차원을 빼면 익명 처리된 쿼리까지 합계에 잡혀서 페이지 합계가 더 정확함
- `inspections(url, checked_at, verdict, coverage_state, google_canonical, last_crawl)`
- 구글은 16개월치만 보여주므로 로컬에 누적해야 전년 동기 비교가 된다

### 분석 레시피 (`gsc-insights` 스킬)

블로그 규모가 작은 초기에는 기준값을 낮게 잡고, 데이터가 쌓이면 올린다.

| 분석 | 조건 (초기값) | 행동 |
|---|---|---|
| 미색인 새 글 | 발행 14일 지나도 색인 안 됨 | 내부 링크 추가, 품질·중복 점검 |
| 제목 개선 | 최근 28일 노출 100 이상, CTR이 같은 순위대 자체 평균의 절반 이하 | 제목·첫 문단 리라이트 |
| 순위 끌어올리기 | 평균 순위 8~20위, 노출 50 이상 | `/refresh-post`로 빠진 항목·표·FAQ 보강 |
| 새 주제 발굴 | 노출은 있는데 매칭 페이지 순위가 20위 밖이거나 의도가 다른 쿼리 | `backlog.csv` 후보로 추가 |
| 자기 잠식 | 같은 쿼리에 내 글 2개 이상이 번갈아 노출 | 통합하거나 의도를 분리 |
| 하락 감지 | 최근 28일 클릭이 이전 28일보다 30% 이상 감소 | 원인 점검 (제도 변경, 경쟁 글, 색인 문제) |

### 파이프라인 연결 지점

- ⓪ 주제 기획: 새 주제 발굴 결과가 후보 1순위
- ① 브리프: 목표 키워드로 이미 노출 중인 내 글이 있으면 새 글 대신 `/refresh-post`
- ⑩ 내부 링크: 관련 쿼리로 노출되는 내 글을 관련 글로 연결
- ⑬ 사후 처리: URL 검사로 색인 확인
- `/weekly-report`: 위 분석 결과 + 지난주 발행 글 성과 + 다음 주 할 일 3가지를 한 장으로

애드센스 수익 데이터는 Search Console에 없다. 1~2개월차는 애드센스 리포트를 월 1회 CSV로 받아 카테고리별 RPM을 보고, 이후 AdSense Management API 연동을 검토한다.

---

## 10. 위험과 안전장치

| 위험 | 안전장치 |
|---|---|
| 애드센스 정책 위반 | 6.1 하드 필터를 4곳에서 검사, 애매하면 탈락 |
| 대량 저품질 판정 | 하루 1편(최대 2편), 1차 정보 1개 이상, 사람 승인, 품질 등급 기록 |
| 사실 오류 | 1등급 출처 규칙, ⑥ 검증, 윤문 후 재대조, 기준일 표기 |
| 저작권 (문장·이미지·폰트) | 연속 일치 검사, 이미지 라이선스 기록, OFL 폰트만, 직접 찍은 사진 우선 |
| AI 이미지 오용 | 사실 정보는 템플릿 이미지로, AI 일러스트는 분위기용 0~1장, 글자·실존 인물·로고 금지, 생성 메타데이터 기록 |
| 개인정보 노출 | 스크린샷 가림 처리, EXIF 위치정보 제거 |
| 무효 트래픽 | 본인 광고 클릭 금지, 트래픽 교환·자동 방문 금지 |
| 키 유출 | 서비스 계정 키·API 키는 저장소 밖 또는 환경 변수, `.env.example`만 커밋 |
| im-not-ai 과윤문·의미 변형 | 스킬 게이트 + 우리 쪽 H2·수치 재검사, 실패 시 초안으로 되돌림 |
| 경험 위조 | `02_experience.md`에 없는 경험은 쓰지 않음 (CLAUDE.md 규칙) |

im-not-ai 설명에 "AI detector bypass" 같은 트리거가 있지만, 이 계획은 읽기 좋은 문장을 만드는 용도로만 쓴다. 구글은 AI 판별기 점수로 순위를 매기지 않으며, 저가치 글의 문체만 바꾸는 전략은 통하지 않는다.

---

## 11. 로드맵

| 기간 | 할 일 | 완료 기준 |
|---|---|---|
| 0주차 (2~3일) | private GitHub 저장소 생성, Search Console API 설정과 16개월 백필, 네이버 검색광고·데이터랩·Gemini API 준비(4.7), 6.2 후보 30개 검색량 조회로 첫 4주 backlog 확정, 정책 원문 대조 후 `adsense-topic-policy.yaml` 작성, 티스토리 호환성 점검(7장), 블로그 단위 기술 점검(6.3), 본인 글 3편으로 `style-guide.md` 작성 | 설정 파일, `gsc.sqlite`, 첫 4주 backlog 준비 완료 |
| 1주차 | CLAUDE.md, im-not-ai 설치, 서브에이전트 8종, `/new-post` ①~⑧, `md_to_tistory.py`, `build_preview.py` | 이미지 없이 3편 발행, 글당 시간·사용량 실측 |
| 2주차 | `image-studio`(썸네일 템플릿 2종, 요약 카드, 흐름도, 차트, Gemini 일러스트), `qa_check.py`, 내부 링크, `gsc_inspect.py` | 이미지 포함 5편 발행, QA 1회 통과율과 이미지 비용 측정 |
| 3주차 | `/plan-week`(네이버 키워드·데이터랩 연동, 점수식), `/weekly-report`, `skill-creator`로 핵심 스킬 평가 | 주 5편, 글당 사람 작업 15분 이하 |
| 4~6주차 | 전환 조건 충족 시 2단계: 네트워크·비밀값 설정, 클라우드 예약 작업, PR 검토 흐름 | 예약 생성 1주 무사고 운영 |
| 2개월차 | `/refresh-post`, 리라이트 주 2편, 시즌 캘린더에 맞춘 선발행 | 첫 리라이트 성과 확인 |
| 3개월차 | 카테고리별 RPM으로 A:B 비중 재조정, AdSense API 연동 검토 | KPI 점검 후 다음 분기 계획 |

---

## 12. `/new-post` 스킬 뼈대

```markdown
---
name: new-post
description: 키워드 하나로 티스토리 글 한 편을 검토 대기 상태까지 만든다. "/new-post 키워드" 또는 "새 글 써줘".
---

0. scripts/gsc_sync.py --incremental 을 실행한다.
1. posts/{날짜}-{slug}/ 를 만들고 serp-analyst로 00_brief.md를 쓴다.
   - adsense-policy-check로 주제를 판정한다. BLOCK·AVOID면 멈추고 이유를 보고한다.
   - gsc_query.py로 같은 키워드에 노출 중인 내 글이 있으면 /refresh-post를 제안하고 멈춘다.
2. researcher로 01_research.md를 만든다 (1등급 출처 규칙).
3. 경험 질문 3개와 필요한 사진 목록을 보여 주고 02_experience.md, images/raw/를 받는다.
   예약 실행이면 이미 있는 파일만 쓰고, 없으면 "경험 없음"으로 진행한다.
4. writer로 03_outline.md → 04_draft.md (seo-structure, style-guide, quick-rules 적용).
5. fact-checker로 05_factcheck.md, 불일치 0건이 될 때까지 수정.
6. policy-reviewer로 06_policy.md. BLOCK·AVOID가 나오면 해당 부분을 빼고 5부터 다시.
7. 아래 두 작업을 병렬로:
   a. /humanize-korean 장르: 블로그 (헤딩·표 구조 유지) → 07_humanized.md, 게이트 결과 기록
   b. visual-designer로 08_image_plan.json → render_template.py / make_chart.py / annotate.py
      / (필요 시) gen_illustration.py → optimize_images.py
8. seo-editor + tistory-format으로 post.html, meta.json, preview.html.
9. qa_check.py. 실패 항목은 해당 단계로 돌아가 고치고 다시 검사 (최대 2회).
10. 검토 대기함에 올리고, 사람이 볼 점(확인 필요 표시, 게이트 경고, 경험 보강 권장)을 요약 보고한다.
```

---

## 13. 결정 내역과 0주차 준비물

### 결정 내역

| 항목 | 결정 |
|---|---|
| 발행 방식 | A방식 (HTML 붙여넣기) |
| 성과 분석 | 처음부터 Search Console API |
| 실행 위치 | 로컬에서 시작 → 전환 조건 충족 시 클라우드 예약 생성 + PC 발행 (8장) |
| 주력 카테고리 | A 생활 돈 관리·제도 활용 60% / B 목표 달성·자기계발 도구 40% (6.2) |
| 일러스트 생성 | Gemini API, Google AI Pro 월 크레딧 사용, 2주차 도입 (4.7) |
| 저장소 | 새 private GitHub 저장소 (예: `lkj1017/tistory-agent`) |
| 제휴 수익 | 하지 않음 |
| 키워드 데이터 | 네이버 검색광고 API + 데이터랩 API + Search Console API |

### 0주차에 준비해 주실 것

1. GitHub private 저장소 (직접 만들거나, 만들어 달라고 요청)
2. Search Console 서비스 계정 연결 (9장 절차)
3. 네이버 검색광고 API 라이선스: 고객 ID, 액세스 라이선스, 비밀키
4. 네이버 개발자센터 애플리케이션: 데이터랩 Client ID·Secret
5. Google Developer Program 혜택 활성화와 Gemini API 키
6. 본인이 직접 쓴 글 3편 (말투 가이드용)
7. 기존 글 목록과 카테고리 구성 (허브 연결용)

키 값은 채팅이나 저장소에 붙여넣지 말고, 로컬 환경 변수나 클라우드 환경 변수에 직접 등록한다.

---

## 참고

정책·SEO (공식 문서, 0주차에 원문 재확인)
- Google 게시자 정책: https://support.google.com/adsense/answer/10502938
- Google 게시자 제한사항: https://support.google.com/adsense/answer/10437795
- 게시자 정책 고객센터: https://support.google.com/publisherpolicies
- 계정 해지로 이어지는 주요 무효 트래픽 및 정책 위반: https://support.google.com/adsense/answer/2660562?hl=ko
- FAQ·HowTo 리치 결과 변경 (2023-08): https://developers.google.com/search/blog/2023/08/howto-faq-changes
- Search Console API 사용 한도: https://developers.google.com/webmaster-tools/limits

도구·플랫폼
- im-not-ai: https://github.com/epoko77-ai/im-not-ai (검토 시점 커밋 `b37e020`, v2.3.2)
- Google AI Pro·Ultra 구독자 개발자 혜택 발표 (2026-01-27): https://blog.google/innovation-and-ai/technology/developers-tools/gdp-premium-ai-pro-ultra/
- Google Developer Program 요금제·혜택: https://developers.google.com/program/plans-and-pricing
- Gemini API 가격표 (0주차 확인): https://ai.google.dev/gemini-api/docs/pricing
- 네이버 검색광고 API 문서: https://naver.github.io/searchad-apidoc/
- 네이버 데이터랩 검색어 트렌드 API: https://developers.naver.com/docs/serviceapi/datalab/search/search.md
- 클라우드 환경 네트워크 설정: https://code.claude.com/docs/en/cloud-environments#network-access
- 티스토리 Open API 종료 관련 질의응답: https://www.inflearn.com/community/questions/1747799
- 티스토리 JSON-LD 적용 사례: https://jab-guyver.co.kr/7176

2차 자료
- 구글 scaled content abuse 정리: https://ppc.land/scaled-content-abuse/
- AI 콘텐츠와 애드센스 정책 정리(2026): https://perplexityaimagazine.com/?p=7275
- 게시자 정책·제한사항 개요: https://optad360.com/blog/does-your-content-comply-with-google-policies-a-quick-guide-through-google-publisher-policies-and-google-publisher-restrictions/
