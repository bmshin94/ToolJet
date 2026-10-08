# ToolJet 전수조사 & 활용 전략 정리 (한국어)

> 작성일: 2026-10-08
> 작성: Claude Code (Karina 페르소나) · 요청자: mono7594@gmail.com
> 분석 대상 저장소: **https://github.com/bmshin94/ToolJet**
> 원본(업스트림): **https://github.com/ToolJet/ToolJet**
> EE 서브모듈(비공개): https://github.com/ToolJet/ee-server · https://github.com/ToolJet/ee-frontend
> 분석 기준: 브랜치 `claude/intelligent-goldberg-urofu1` / `.version` = `3.21.71-beta` / `package.json` = `1.18.0`

---

## 목차

1. [이게 뭐하는 프로젝트인가](#1-이게-뭐하는-프로젝트인가)
2. [폴더 전수조사 결과](#2-폴더-전수조사-결과)
3. [쉽게 이해하는 ToolJet](#3-쉽게-이해하는-tooljet)
4. [설치 및 사용법](#4-설치-및-사용법)
5. [플러그인? 스킬? MCP?](#5-플러그인-스킬-mcp)
6. [API 토큰 사용 여부](#6-api-토큰-사용-여부)
7. [AI 에이전트 구축에 도움이 되는가](#7-ai-에이전트-구축에-도움이-되는가)
8. [React / PHP로 만들 수 있는가](#8-react--php로-만들-수-있는가)
9. [유튜브 강의 제작 가능성](#9-유튜브-강의-제작-가능성)
10. [수익화 아이디어 완전판](#10-수익화-아이디어-완전판)
11. [90일 실행 플랜](#11-90일-실행-플랜)
12. [리스크 체크리스트](#12-리스크-체크리스트)
13. [참고 링크](#13-참고-링크)

---

## 1. 이게 뭐하는 프로젝트인가

**ToolJet = 사내 업무용 앱(어드민 패널 / 대시보드 / 운영 도구)을 드래그앤드롭 + AI 프롬프트로 만드는 오픈소스 로우코드 플랫폼.**

쉽게 말해 **Retool의 오픈소스 대안**. Retool은 월 수백만 원대 과금이지만 ToolJet은 AGPL-3.0으로 셀프호스팅이 가능하다.

### 핵심 가치

기존 어드민 개발에서 가장 오래 걸리는 작업(로그인·세션, 권한 관리, 테이블 컴포넌트, API 서버, DB 연결, 배포/CI)을 ToolJet이 이미 전부 갖고 있다. 개발자는 "조립"만 한다.

| 작업 | 전통 개발 | ToolJet |
|---|---|---|
| 프로젝트 셋업 | 1일 | 0 (Docker 5분) |
| 로그인/세션 | 2일 | 0 (내장) |
| 권한 관리 (RBAC) | 3일 | 0 (내장, CASL) |
| 테이블(정렬/필터/페이징) | 3일 | 0 (컴포넌트 드롭) |
| 백엔드 API | 5일 | 0 (쿼리 패널) |
| DB 연결 | 2일 | 3분 |
| 배포/CI | 2일 | 0 (Docker/K8s 준비됨) |
| **합계** | **3~4주** | **수십 분 ~ 며칠** |

### 쓰기 좋은 상황

| 상황 | ToolJet으로 하면 |
|---|---|
| 고객 CS팀 조회/수정 화면 | DB 연결 → Table 드롭 → 30분 |
| 주문 환불 처리 어드민 | 쿼리 + 버튼 이벤트 → 1시간 |
| 매출 대시보드 | Chart + SQL → 30분 |
| 엑셀로 돌리던 재고관리 | ToolJet DB + Form → 반나절 |
| 승인 워크플로우 | Workflows (분기/스케줄/웹훅) |
| 사내 AI 챗봇 | Anthropic/OpenAI 플러그인 + Pinecone(RAG) + Chat 컴포넌트 |

### 안 맞는 경우

B2C 서비스, 픽셀 단위 디자인이 중요한 화면, 고트래픽 공개 웹사이트.

---

## 2. 폴더 전수조사 결과

```
/home/user/ToolJet/  (v3.21.71-beta, Node 22.15.1, npm 10.9.2)
├── frontend/     864MB  React 앱빌더 (JS/JSX 2,523개 파일)
├── docs/         689MB  Docusaurus 공식 문서 사이트
├── server/       100MB  NestJS 백엔드 (TS 1,205개 파일, 60+ 모듈)
├── marketplace/   14MB  서드파티 플러그인 45개
├── cypress-tests/  6MB  E2E 테스트 (happyPath)
├── plugins/      4.7MB  내장 커넥터 49개 (Lerna 모노레포)
├── cli/          372KB  @tooljet/cli v0.0.14 (플러그인 생성 도구, oclif)
├── deploy/       212KB  docker / ec2 / helm / kubernetes / openshift
├── docker/       196KB  에디션별 Dockerfile
├── scripts/       40KB  coverage-gate, test-changed, sync-skills 등
├── release-scripts/
├── queryPanel/          쿼리 패널 회귀 테스트 2개
├── plans/               widget-css-class.md
├── .claude/             Claude Code 스킬 16개 + 서브에이전트 3개
└── .agents/             에이전트용 컨텍스트 맵 (product/architecture) + 스킬 원본
```

### 2.1 frontend/ — 비주얼 앱 빌더

`frontend/src/AppBuilder/WidgetManager/widgets/` 에 **83개 컴포넌트** 정의 파일.

| 분류 | 컴포넌트 |
|---|---|
| 데이터 표시 | table, chart, listview, kanbanBoard, timeline, statistics, treeSelect, jsonExplorer, pagination |
| 입력 | textinput, numberinput, datepickerV2, datetimepickerV2, daterangepicker, dropdownV2, multiselectV2, filepicker, fileinput, richtextarea, codeEditor, jsonEditor, phoneinput, currencyinput, emailinput, passwordinput, colorPicker, rangesliderV2, starrating, tagsInput, keyValuePair, cascader |
| 레이아웃 | container, flexContainer, boundedBox, tabs, modalV2, steps, accordion, form, divider, verticalDivider, navigation, popoverMenu |
| 특수 | camera, qrscanner, audioRecorder, map, pdf, chat, iframe, html, svgImage, customComponent, timer, spinner, circularProgressbar, moduleContainer, moduleViewer, reorderableList |

주요 디렉토리:

- `AppBuilder/AppCanvas/` — 드래그앤드롭 캔버스
- `AppBuilder/WidgetManager/` — 컴포넌트 레지스트리
- `AppBuilder/QueryManager/` — 쿼리 패널
- `AppBuilder/_stores/` — zustand 기반 런타임 상태
- `AppBuilder/Viewer/Viewer.jsx` — 릴리스된 앱 렌더러
- `TooljetDatabase/` — 내장 노코드 DB UI
- `WorkflowEditor/` — 워크플로우 비주얼 에디터
- `modules/common/helpers/_registry/` — 에디션별 모듈/컴포넌트 레지스트리

### 2.2 server/ — NestJS 백엔드

`server/src/modules/` 주요 모듈:

| 모듈 | 역할 |
|---|---|
| `apps/`, `versions/`, `app-history/` | 앱 CRUD + 버전 관리 + 히스토리 |
| `data-sources/`, `data-queries/`, `data-query-folders/` | 데이터소스 연결 + 쿼리 실행 |
| `tooljet-db/` | 내장 PostgreSQL DB (PostgREST 프록시) |
| `ai/` | AI 앱생성 — `agents.service.ts`, `graph.service.ts`, `ai-cache.ts` |
| `workflows/` | 워크플로우 엔진 — `agent-node.service.ts`, Python/JS 샌드박스 |
| `personal-access-tokens/` | API 토큰 (PAT) — CLI/MCP 인증 |
| `external-apis/` | 외부 공개 REST API |
| `git-sync/`, `app-git/`, `platform-git-sync/`, `git-sync-webhooks/` | GitSync (앱을 Git으로 버전관리) |
| `group-permissions/`, `casl/`, `ability/`, `roles/` | RBAC 권한 (CASL 기반) |
| `scim/`, `login-configs/`, `white-labelling/`, `custom-domains/` | 엔터프라이즈 (SCIM, SSO, 화이트라벨) |
| `licensing/` | 라이선스 검증 (CE/EE 기능 게이팅) |
| `audit-logs/` | 감사로그 (규제 산업 필수) |
| `encryption/` | AES-256-GCM 암호화 |
| `modules/` | 재사용 가능한 UI/로직 단위 (EE) |
| `import-export-resources/` | 앱 JSON export/import |
| `organizations/`, `organization-users/`, `organization-constants/` | 워크스페이스 관리 |

보조 디렉토리: `migrations/` (TypeORM 스키마), `data-migrations/` (데이터 마이그레이션), `templates/`, `otel/` (OpenTelemetry), `test/`.

### 2.3 plugins/packages/ — 내장 커넥터 49개

```
RDB/DW   PostgreSQL, MySQL, MariaDB, MSSQL, OracleDB, IBM DB2, SAP HANA,
         Snowflake, BigQuery, Athena, Databricks, ClickHouse, Presto 계열
NoSQL    MongoDB, CosmosDB, DynamoDB, CouchDB, RethinkDB, Firestore, Redis
검색/시계열 Elasticsearch, InfluxDB, Typesense
스토리지  S3, GCS, MinIO, Azure Blob
노코드DB  Airtable, Baserow, NocoDB, Notion, Google Sheets (v1/v2)
API      REST API, GraphQL, gRPC (v1/v2), OpenAPI
결제/통신 Stripe, Twilio, SendGrid, Mailgun, Amazon SES, SMTP, Plivo
협업     Slack, Zendesk
커머스/자동화 WooCommerce, n8n, Appwrite
```

### 2.4 marketplace/plugins/ — 서드파티 45개

```
LLM        anthropic, openai, gemini, mistral_ai, cohere, aws-bedrock,
           hugging_face, portkey
벡터DB     pinecone, qdrant, weaviate          ← RAG 구축 가능
SaaS/CRM   salesforce, hubspot, jira, asana, clickup, intercom, quickbooks,
           xero, servicenow, github, gmail, googlecalendar, sharepoint,
           microsoft_graph, supabase, couchbase, pocketbase, harperdb
클라우드    aws-lambda, awsredshift, textract, spanner, s3, azurerepos
모니터링    prometheus, presto
물류/결제   fedex, ups, easypost, aftership, authorizenet
알림       engagespot, plivo
```

**한국 서비스 커넥터는 단 하나도 없다** → 선점 기회.

### 2.5 아키텍처 — 3개 에디션 체계

`TOOLJET_EDITION` 환경변수로 `ce` / `ee` / `cloud` 분기.

- **백엔드 = 상속 패턴**
  - `SubModule` 베이스 클래스 (`server/src/modules/app/sub-module.ts`)
  - `getImportPath()` (`server/src/modules/app/constants/index.ts`) 가 `src/modules/`(CE) 또는 `ee/`(EE/Cloud)로 라우팅
  - EE 서비스가 CE 서비스를 `extends` 하고 `super()` 호출
  - `getTooljetEdition()` (`server/src/helpers/utils.helper.ts`) 런타임 체크
- **프론트엔드 = 합성 패턴**
  - 웹팩 alias: `@/` → `src/`, `@ee/` → `ee/`, `@cloud/` → `cloud/`
  - `NormalModuleReplacementPlugin` 이 하위 에디션에서 `@ee/` import를 빈 모듈로 치환 → 컴파일 타임 격리
  - 런타임 레지스트리로 에디션별 모듈/컴포넌트/스토어 해석
- **Cloud는 별도 코드트리가 아니다** — EE 서브모듈 코드를 그대로 쓰고 런타임 체크로 클라우드 전용 동작만 분기
- EE 코드는 **private 서브모듈** (`server/ee`, `frontend/ee`) → 공개 클론에서는 빈 폴더 (현재 체크아웃 상태 확인됨)

### 2.6 .claude/ + .agents/ — AI 에이전트 인프라 (숨은 보물)

```
.claude/skills/ (16개)              ← .agents/skills/ 의 심볼릭 링크
├── commit, create-pr, merge        루트 + 서브모듈 2개 동시 git 작업
├── bug-triage, codebase-question   디버깅 / 코드베이스 질문
├── app-builder-feature             컴포넌트 기능 추가
├── app-builder-bug-fix
├── app-builder-test-driven-development
├── app-builder-widget-backfill
├── app-builder-grill-me            코드 이해도 퀴즈
├── api-design, create-issue
├── manage-dependabot-alerts, page-load-audit
└── manage-skills

.claude/agents/ (3개 서브에이전트)
├── component-config-reader.md   컴포넌트 config에서 테스트 surface 추출 (YAML)
├── facet-coder.md               Cypress 스펙 자동 생성 (facet 단위)
└── helper-author.md             Cypress 헬퍼 라이브러리 관리 + 인덱스 재생성

.agents/context/
├── product-map.md       제품 지도 (유저/역할/여정/확인된 비즈니스 룰/추론)
└── architecture-map.md  시스템 지도 (런타임/데이터스토어/인증/통합/배포/장애모드)
```

`scripts/sync-skills.sh` 가 심볼릭 링크를 정리하고, pre-commit 훅이 `--check` 모드로 검증한다.
일부 private 스킬(`bug-triage`, `page-load-audit` 등)은 `frontend/ee` 서브모듈에 실체가 있어 EE 접근 권한 없는 클론에서는 링크가 끊긴다.

**이것은 ToolJet 팀이 자기 코드베이스를 AI로 개발하기 위해 만든 에이전트 인프라**이며, Claude Code 스킬 작성법의 실전 레퍼런스로서 독립적인 가치가 있다.

### 2.7 문서 체계 (계층형 컨텍스트)

가장 가까운 파일이 우선한다.

| 파일 | 범위 |
|---|---|
| `AGENTS.md` (루트, `CLAUDE.md`는 심볼릭 링크) | 저장소 전체 아키텍처, 에디션, 구조 |
| `.agents/context/product-map.md` | 공개 제품 기능, 유저, 여정, 비즈니스 룰 |
| `.agents/context/architecture-map.md` | 시스템 구성요소, 데이터 흐름, 통합, 장애모드 |
| `UBIQUITOUS_LANGUAGE.md` | 도메인 용어 사전 (21KB) |
| `server/AGENTS.md` | 백엔드 + 테스트 규약 |
| `server/src/modules/<module>/AGENTS.md` | 모듈별 목적, 핵심 파일, 불변식 |
| `frontend/AGENTS.md` | 프론트 규약, 앱빌더 아키텍처, 용어 |
| `server/docs/testing.md` | 백엔드 테스트 가이드 |

### 2.8 용어 함정 (UBIQUITOUS_LANGUAGE.md)

- **Organization**(코드) = **Workspace**(사용자 노출) — 같은 개념
- **Widget**(레거시) = **Component**
- **Module** 이 중복 사용됨: NestJS 모듈(백엔드) vs 재사용 앱 블록(프론트/EE 기능)
- `data_source` 를 `ds` 로 줄여 쓰지 않는다

### 2.9 Workspace 모델

```
Instance (서버 1대)
└── Workspace A (테넌트 경계)
    ├── 유저 & 그룹 (권한)
    ├── 앱들 → 버전 → 페이지 → 컴포넌트 → 이벤트/액션
    ├── 데이터소스
    ├── ToolJet Database (내장 테이블)
    └── 상수 / 시크릿
└── Workspace B  ← A의 리소스에 접근 불가
```

### 2.10 확인된 보안 설계 (product-map.md 기준)

- Workspace 가 앱·유저·데이터소스·상수·ToolJet DB 테이블의 테넌트 경계
- 권한은 **서버 측 CASL ability guard** 로 강제. UI 가시성은 인증 경계가 아니다
- 모든 백엔드 기능 라우트는 module/feature 메타데이터 + ability guard 선언 필수.
  `GuardValidator` 가 부팅 시 검사 (`server/src/modules/app/validators/feature-guard.validator.ts`)
- 앱 콘텐츠는 Version 에 귀속. 앱 이름/슬러그/아이콘/공개여부가 version 행에 저장
- 엔드유저는 릴리스된 버전을 소비. 공개 앱은 로그인을 우회하지만 쿼리 실행은 여전히 guard + throttle 적용
- 쿼리 자격증명과 상수는 선택된 Environment 기준으로 **서버에서 복호화** — 브라우저로 노출되지 않음
- 컴포넌트 설정은 기존 저장된 앱과 하위호환을 유지해야 함

### 2.11 요청 처리 흐름 (예: "주문 환불" 버튼 클릭)

```
[브라우저] 버튼 클릭
   ↓
[프론트] {{ }} 표현식 해석 → orderId = 1234
   ↓
[서버] AuthGuard('jwt') → tj_auth_token 쿠키 검증
   ↓
[서버] AbilityGuard (CASL) → 환불 권한 확인
   ↓
[서버] DataQueriesUtilService.runQuery
       → 현재 환경(dev/stage/prod)의 DB 자격증명 복호화
   ↓
[플러그인] plugins/packages/postgresql → 실제 쿼리 실행
   ↓
[서버] audit-logs 기록
   ↓
[브라우저] 결과 수신 → 테이블 갱신
```

### 2.12 CE vs EE 기능 비교

| 기능 | CE (무료) | EE (유료) |
|---|---|---|
| 비주얼 앱 빌더 (83개 컴포넌트) | O | O |
| 데이터소스 90+ | O | O |
| ToolJet Database | O | O |
| 멀티페이지 앱 / 멀티플레이어 편집 | O | O |
| 셀프호스팅 (Docker/K8s/Helm) | O | O |
| JS/Python 코드 실행 | O | O |
| 인라인 댓글 / 멘션 | O | O |
| AES-256-GCM 암호화, SSO | O | O |
| **AI 앱 생성 (프롬프트 → 앱)** | X | O |
| **AI 쿼리 빌더 / AI 디버깅** | X | O |
| **Agent Builder** | X | O |
| **Workflows (분기/스케줄/웹훅)** | X | O |
| **Modules (재사용 UI/로직)** | X | O |
| RBAC 세밀권한 / 커스텀 그룹 / SCIM | X | O |
| GitSync / CI-CD / 버전 히스토리 | X | O |
| 멀티환경 (dev/stage/prod) | X | O |
| 화이트라벨 / 커스텀 도메인 / 테마 | X | O |
| 행·컴포넌트·페이지·쿼리 단위 접근제어 | X | O |
| 임베디드 앱 | X | O |
| 감사로그 / SOC2·GDPR 대응 | X | O |
| 엔터프라이즈 지원 (SLA) | X | O |

> CE 에서 workflow 웹훅 메서드 일부는 stub 이다 (product-map.md 명시).

### 2.13 라이선스 (AGPL-3.0) — 중요

| 하는 일 | 가능 여부 |
|---|---|
| 사내에서 사용 | 가능 |
| 고객 어드민 구축해주고 비용 수령 (앱은 고객 자산) | 가능 |
| 강의 / 유튜브 / 블로그 콘텐츠 제작 | 가능 |
| 배포·운영·호스팅 서비스 제공 (코드 수정 없음) | 가능 |
| 코드 수정해서 **SaaS로 서비스** | **수정분 소스 공개 의무** |
| 코드 수정해서 사내 전용 사용 | 가능 |

SaaS 재판매를 원하면 ToolJet 측 상용 라이선스 문의 필요.

---

## 3. 쉽게 이해하는 ToolJet

### 비유: 맥도날드 주방

일반 앱 개발이 "소 키우고 밭 갈아서 햄버거 만들기"라면, ToolJet 은 "손질된 재료와 조립 라인이 깔린 주방"이다.

### ToolJet 의 4대 부품

| 부품 | 비유 | 설명 | 코드 위치 |
|---|---|---|---|
| **컴포넌트** | 레고 블록 | 화면에 보이는 것 (버튼/표/차트/입력) 83개 | `frontend/src/AppBuilder/WidgetManager/widgets/table.js` |
| **데이터소스** | 전기 콘센트 | "내 데이터가 어디 있는지" 등록 | `plugins/packages/postgresql/` |
| **쿼리** | 주문서 | "이 데이터소스에서 이걸 가져와" | `server/src/modules/data-queries/` |
| **이벤트/액션** | 스위치 | "클릭하면 → 쿼리 실행 → 알림" | `server/src/modules/apps/services/event.service.ts` |

### `{{ }}` 표현식 — ToolJet 의 핵심 마법

```sql
SELECT * FROM orders WHERE status = {{ components.dropdown1.value }}
```

컴포넌트 값, 쿼리 결과, 전역변수, 상수, 시크릿을 서로 꽂을 수 있다.
해석 파이프라인은 `frontend/AGENTS.md` 에 문서화되어 있다.

### 데이터소스 비밀번호는 안전한가

안전하다. AES-256-GCM 으로 암호화되어 저장되고, 복호화는 **서버에서만** 일어난다. 브라우저로 자격증명이 전달되지 않는다.

---

## 4. 설치 및 사용법

### 방법 A: 맛보기 (5분, 가장 추천)

```bash
docker run \
  --name tooljet \
  --restart unless-stopped \
  -p 80:80 \
  --platform linux/amd64 \
  -v tooljet_data:/var/lib/postgresql/13/main \
  tooljet/try:ee-lts-latest
```

`http://localhost` 접속. Postgres 포함 올인원 이미지.
주의: 데모용이므로 프로덕션 사용 금지 (DB가 컨테이너 내부).

> 업그레이드 시에는 latest 보다 **LTS 버전**을 권장한다 (프로덕션 버그픽스·보안패치·성능개선 포함).

### 방법 B: ToolJet Cloud (설치 0분)

https://tooljet.com 가입 후 즉시 사용. 호스팅형.

### 방법 C: 로컬 개발 환경 (코드 분석용)

```bash
# 1. 노드 버전 고정 (필수 — engines: node 22.15.1 / npm 10.9.2)
nvm use                      # .nvmrc = v22.15.1

# 2. 환경변수
cp .env.example .env
#   LOCKBOX_MASTER_KEY  (64자 hex)  ← 데이터소스 암호화 마스터키
#   SECRET_KEY_BASE                 ← 세션 서명
#   MFA_MASTER_SECRET               ← 2FA
#   PG_DB=tooljet_ce / PG_USER / PG_HOST / PG_PASS

# 3. 플러그인 먼저 빌드 (중요 — db:migrate 가 @tooljet/plugins/dist/server 에 의존)
cd plugins && npm install && npm run build && cd ..

# 4. 루트 install (husky 훅 활성화)
npm install

# 5. DB 셋업
cd server && npm run db:setup

# 6. 기동 (터미널 2개)
cd server && npm run start:dev      # .env 의 PORT
cd frontend && npm start            # 8082
```

### 방법 D: Docker Compose (개발용, 가장 편함)

```bash
cp .env.example .env
docker-compose up
```

기동되는 컨테이너 6개:

| 컨테이너 | 포트 | 역할 |
|---|---|---|
| `plugins` | — | 플러그인 watch 빌드 |
| `client` | 8082 | 프론트 dev server |
| `server` | 3000 | NestJS |
| `postgres` | 5432 | 메인 DB (postgres:13) |
| `redis` | 6379 | 큐/캐시 (redis:7-alpine) |
| `postgrest` | 3001 | ToolJet DB 프록시 (postgrest v12.2.0) |

### 방법 E: 프로덕션 배포

`deploy/` 에 `docker/`, `ec2/`, `helm/`, `kubernetes/`, `openshift/` 준비됨.
공식 가이드 14종: DigitalOcean, Docker, AWS EC2/ECS/EKS, GCP GKE/Cloud Run, Azure AKS/Container, OpenShift, Helm, 클라이언트 단독 배포, 서브패스 배포.

### 자주 쓰는 명령어

```bash
npm run db:migrate            # 마이그레이션
npm run db:seed               # 시드
npm run db:reset              # DB 초기화
npm run db:drop               # DB 삭제
npm run build                 # plugins:prod + frontend + server 전체 빌드
npm run start:prod            # 프로덕션 기동
npm run worker:prod           # 워커 프로세스
npm run plugins:install       # 플러그인 설치/언인스톨/리로드
npm run reset-superadmin      # 슈퍼관리자 리셋
npm run reset-mfa             # MFA 리셋
npm run rotate:keys           # 암호화 키 로테이션
npm run ci:changed            # 변경분만 테스트 (scripts/test-changed.sh)
cd server && npm test         # 백엔드 테스트
cd server && npm run lint     # 백엔드 린트 (pre-commit 훅은 프론트만 처리)
```

> 린트 주의: husky + lint-staged 는 **프론트엔드 파일만** 자동 수정한다. 백엔드는 `cd server && npm run lint` 를 수동 실행해야 한다. CI는 세 폴더 전부 린트하고 실패 시 PR을 막는다.

### 실제 앱 만드는 순서

```
1. 로그인 → Workspace 진입
2. "Create new app"
3. 좌측 컴포넌트 패널에서 Table 드래그 → 캔버스 드롭
4. 하단 Query Panel → "+ Add" → PostgreSQL 선택
5. 데이터소스 미등록이면 → Data sources → 접속정보 입력
6. SQL 작성 → Preview 로 결과 확인
7. Table 선택 → 우측 Inspector → Data 속성에 {{queries.query1.data}}
8. 버튼 추가 → Events → On click → Run query
9. 우상단 "Release" → /applications/<slug> 로 공개
```

### 주요 환경변수 (.env.example 기준)

```bash
TOOLJET_HOST=http://localhost:8082
LOCKBOX_MASTER_KEY=<64자 hex>     # 데이터소스 암호화 마스터키 — 유실 시 복구 불가
SECRET_KEY_BASE=<랜덤>
MFA_MASTER_SECRET=<긴 랜덤>
PG_DB / PG_USER / PG_HOST / PG_PASS
TOOLJET_DB / TOOLJET_DB_USER / TOOLJET_DB_HOST / TOOLJET_DB_PASS
TOOLJET_DB_STATEMENT_TIMEOUT=60000
PGRST_HOST / PGRST_JWT_SECRET / PGRST_DB_PRE_CONFIG=postgrest.pre_config
REDIS_HOST / REDIS_PORT / REDIS_PASSWORD / REDIS_TLS
WORKER= / TOOLJET_QUEUE_DASH_PASSWORD= / TOOLJET_WORKFLOW_SANDBOX_BYPASS=
SMTP_* / DEFAULT_FROM_EMAIL
SSO_GOOGLE_OAUTH2_CLIENT_ID / SSO_GIT_OAUTH2_* / SSO_ACCEPTED_DOMAINS
ENABLE_MULTIPLAYER_EDITING=true
COMMENT_FEATURE_ENABLE=
DISABLE_SIGNUPS=
APM_VENDOR / SENTRY_DNS
```

데이터베이스 이름 규약: `tooljet_{edition}`.

---

## 5. 플러그인? 스킬? MCP?

**정답: 세 가지 모두 아니다. 세 가지를 "제공하는" 제품(플랫폼)이다.**

### ToolJet 자체 = 풀스택 웹 애플리케이션

NestJS + React + PostgreSQL + Redis 가 돌아가는 독립 서비스. 무언가에 끼워 넣는 부품이 아니다.

### 그런데 ToolJet 이 가진 3종 세트

#### ① "플러그인" — ToolJet 내부의 커넥터 시스템

```
plugins/packages/*     내장 커넥터 49개
marketplace/plugins/*  서드파티 45개
```

`@tooljet/cli` 로 직접 제작 가능:

```bash
npx @tooljet/cli plugin create my-connector
```

이는 **ToolJet 의 플러그인**이며, Claude Code 플러그인이 아니다.

#### ② "스킬" — 이 저장소 안의 Claude Code 스킬 (`.claude/skills/`)

ToolJet 을 **개발하는 사람/AI 를 위한** 16개 스킬. ToolJet 을 "쓰는" 스킬이 아니라 ToolJet 을 "만드는" 스킬이다.
이 저장소에서 Claude Code 를 실행하면 `/commit`, `/create-pr`, `/merge` 등을 바로 사용할 수 있다.

#### ③ "MCP" — ToolJet MCP 서버 (베타)

README 명시:

> ToolJet ships a Model Context Protocol server, so the coding agent you already use can build ToolJet apps directly: generating pages, queries, and components from a prompt, and modifying existing apps in place.

| 클라이언트 | 연결 방식 |
|---|---|
| Claude Code / Codex / Grok Build | 플러그인 형태 (앱빌더 스킬 번들 포함) |
| Cursor 및 기타 MCP 클라이언트 | MCP 서버 단독 연결 |

핵심 포인트 두 가지:

1. 에이전트가 **ToolJet 의 실제 컴포넌트·데이터 계약(governed first-party contracts)** 에 대해 빌드한다. 자유형식 코드를 뱉는 게 아니라 실제 ToolJet 앱이 산출되고, 팀은 비주얼 빌더에서 계속 편집할 수 있다. 권한·환경·버전 히스토리가 그대로 적용된다.
2. **내 모델 구독으로 동작** → ToolJet AI 크레딧이 소모되지 않는다.

설정 가이드: https://docs.tooljet.com/docs/build-with-ai/mcp/overview

### 한 장 정리

| 질문 | 답 |
|---|---|
| ToolJet 은 플러그인? | 아니다. 독립 제품/플랫폼 |
| ToolJet 은 스킬? | 아니다. 단, 저장소에 **개발용 Claude 스킬 16개 내장** |
| ToolJet 은 MCP? | 아니다. 단, **MCP 서버를 제공**해 AI 에이전트가 ToolJet 앱을 생성 |
| 플러그인을 가지는가? | 그렇다. 커넥터 94개 (내장 49 + 마켓 45) |

---

## 6. API 토큰 사용 여부

상황에 따라 다르다. 4가지 케이스.

### 케이스 1: 브라우저로 앱 만들 때 → 토큰 불필요

로그인하면 `tj_auth_token` 쿠키로 세션 관리.

### 케이스 2: 프로그램/CLI/MCP 로 ToolJet 조작 → PAT 필요

구현: `server/src/modules/personal-access-tokens/`

```ts
export const PAT_TOKEN_PREFIX = 'tj_pat_';
export const PAT_API_SOURCE = 'personal_access_token';
```

| 특징 | 내용 |
|---|---|
| 접두사 | `tj_pat_...` |
| 소유 구조 | **유저 소유 + 워크스페이스 바인딩** (한 토큰 = 한 워크스페이스) |
| 만료 | **반드시 만료** (유저가 미래 날짜 지정, 무기한 불가) |
| 동작 원리 | 토큰 자체가 자격증명이 아니라 **짧은 세션을 발급받는 티켓** |
| 감사 추적 | 세션 JWT 에 `tj_api_source` 스탬프 → 감사로그에서 사람/봇 구분 |
| 권한 설정 | 코드 상수로 고정 (`PAT_BUNDLE`), 토큰별 커스터마이즈 불가 |
| 킬 스위치 | 토큰 행 삭제 시 그 토큰에서 발급된 모든 세션 무효화 |

#### 권한 번들 (`constants/scopes.ts`)

| 번들 | 포함 모듈 |
|---|---|
| `APPS` | APP, VERSION, APP_HISTORY, APP_PERMISSIONS, APP_GIT, FOLDER, FOLDER_APPS, MODULE_FOLDER, MODULES, IMPORT_EXPORT_RESOURCES, TEMPLATES, COMMENT, THREAD, FILE, ORGANIZATION_THEMES |
| `DATA` | DATA_QUERY, DATA_QUERY_FOLDERS, GLOBAL_DATA_SOURCE, TOOLJET_DATABASE, APP_ENVIRONMENTS |
| `WORKFLOWS` | WORKFLOWS, WORKFLOW_FOLDER |
| `WORKSPACE_USERS` | ORGANIZATION_USER |
| `WORKSPACE_ADMIN` | ORGANIZATIONS, USER, GROUP_PERMISSIONS, ORGANIZATION_CONSTANT, ORGANIZATION_VARIABLE, ORGANIZATION_PAYMENTS, CUSTOM_STYLES, CUSTOM_DOMAINS, WHITE_LABELLING, LOGIN_CONFIGS, GIT_SYNC, GIT_SYNC_CONFIGS, WORKSPACE_BRANCHES, SMTP, CONFIGS |
| `INSTANCE_ADMIN` | INSTANCE_SETTINGS, LICENSING, SCIM, AUDIT_LOGS, METRICS, PLUGINS, CRM |

코드 주석에 설계 의도가 명시되어 있다 (설계 품질이 높은 부분):

> "Deliberately a code-level constant rather than per-token data: every workspace PAT gets the same ceiling, so there is nothing to configure, nothing to migrate, and no way to mint an accidentally-unrestricted token."

> "What a PAT buys: a short-lived SESSION, not access. The internal APIs authenticate a session (a dozen AuthGuard('jwt') subclasses, ~90 controllers), so the token's job is to mint one rather than to become a second credential every guard must learn."

**발급 방법:** Settings → Personal access tokens → 이름 + 만료일 → 생성.
토큰 원문은 **생성 시 한 번만** 표시된다.

**서비스 PAT:** 백엔드 서비스가 로그인 유저를 대행할 때 쓰는 변종(`getOrCreateServicePat`)이 있다. 원문 토큰을 생성 즉시 폐기하므로 bearer 자격증명으로 제시될 수 없고, 세션 앵커 + 킬 스위치 역할만 한다.

### 케이스 3: 외부 시스템이 ToolJet 을 호출 → 외부 API

`server/src/modules/external-apis/controllers/`

| 컨트롤러 | 역할 |
|---|---|
| `apps.controller.ts` | 앱 CRUD |
| `app-export.controller.ts` | 앱 내보내기 |
| `groups.controller.ts` | 권한 그룹 |
| `tooljet-db.controller.ts` | 내장 DB |
| `modules.controller.ts` | 모듈 |
| `ban.controller.ts` | 유저 차단 |

### 케이스 4: 데이터소스/외부서비스 연결 → 해당 서비스 토큰

ToolJet 토큰이 아니라 연결 대상의 키가 필요하다.
예: Anthropic API Key, OpenAI API Key, Slack Bot Token, Pinecone API Key.
모두 AES-256-GCM 으로 암호화 저장되고 서버에서만 복호화된다.

### 반드시 관리해야 할 시크릿

```bash
LOCKBOX_MASTER_KEY=<64자 hex>   # 데이터소스 암호화 마스터키 — 유실 시 모든 접속정보 복구 불가
SECRET_KEY_BASE=<랜덤>          # 세션 서명
MFA_MASTER_SECRET=<긴 랜덤>     # 2FA
PGRST_JWT_SECRET=<랜덤>         # PostgREST JWT
```

`LOCKBOX_MASTER_KEY` 는 반드시 안전하게 백업한다. 로테이션은 `npm run rotate:keys`.

---

## 7. AI 에이전트 구축에 도움이 되는가

### 결론

**매우 도움이 된다. 단, "에이전트의 두뇌"가 아니라 "에이전트의 몸통"으로서.**

LangGraph / CrewAI 같은 프레임워크의 대체재는 아니다. 대신 에이전트 제품화에서 가장 귀찮은 부분을 전부 제공한다.

### ToolJet 이 제공하는 것

| 에이전트에 필요한 것 | ToolJet 이 주는 것 |
|---|---|
| UI | Chat 컴포넌트, 83개 컴포넌트, 실시간 바인딩 |
| 인증/권한 | 로그인, SSO, CASL RBAC, 멀티테넌시 |
| 툴(Tool) | 커넥터 94개 = 에이전트가 호출할 도구 94개 |
| 메모리 / RAG | Pinecone, Qdrant, Weaviate 플러그인 |
| LLM 연결 | Anthropic, OpenAI, Gemini, Mistral, Cohere, Bedrock, HuggingFace, Portkey |
| 오케스트레이션 | Workflows (분기, 스케줄, 웹훅) |
| Human-in-the-loop | 승인 UI 를 드래그앤드롭으로 구성 (킬러 기능) |
| 감사로그 | audit-logs 모듈 (금융/의료/공공 필수) |
| 배포 | Docker / K8s / Helm 준비됨 |

### 실제 코드로 확인된 AI 인프라

```
server/src/modules/ai/
├── services/agents.service.ts                     에이전트 서비스 (EE 구현체는 private)
├── services/graph.service.ts                      그래프 기반 오케스트레이션
├── repositories/ai-conversation.repository.ts     대화 저장
├── repositories/ai-conversation-message.repository.ts  메시지 저장
├── repositories/artifact.repository.ts            산출물 저장
├── repositories/ai-response-vote.repository.ts    응답 피드백 수집
├── schedulers/clear-stale-ai-runs.scheduler.ts    좀비 런 정리
├── ai-cache.ts                                    응답 캐싱
├── interfaces/IAgentsService.ts
└── ability/guard.ts                               권한 가드

server/src/modules/workflows/services/
├── agent-node.service.ts                 워크플로우 내 "에이전트 노드"
├── python-executor.service.ts            Python 샌드박스 실행
├── python-bundle-generation.service.ts
├── bundle-generation.service.ts
├── security-mode-detector.service.ts     보안모드 감지
├── workflow-stream.service.ts            스트리밍
├── workflow-execution-queue.service.ts   BullMQ 큐
├── workflow-scheduler.service.ts         스케줄
├── workflow-termination-registry.ts      중단 처리
├── npm-registry.service.ts / pypi-registry.service.ts
└── schedule-bootstrap.service.ts
```

설계상 **에이전트가 워크플로우의 한 노드가 되는** 구조다.

### 구성 예시

```
┌─────────────────────────────────────────┐
│  ToolJet 앱 (Chat UI + 승인 버튼)         │  드래그앤드롭 10분
└──────────────┬──────────────────────────┘
               ↓
┌─────────────────────────────────────────┐
│  ToolJet Workflow                        │
│  1) Anthropic 플러그인 → 의도 분류         │
│  2) 분기: 조회 / 수정 / 에스컬레이션        │
│  3) Pinecone → 사내문서 RAG 검색           │
│  4) PostgreSQL → 실제 데이터 조회          │
│  5) Human 승인 대기 (수정 작업인 경우)      │
│  6) Slack → 결과 알림                     │
└─────────────────────────────────────────┘
```

### 한계

| 한계 | 설명 |
|---|---|
| Workflows / Agent Builder = EE 유료 | CE 에서는 workflow 웹훅 메서드 일부가 stub |
| 복잡한 에이전트 로직 | 코드로 작성하는 편이 낫다 (LangGraph 등) |
| EE 구현체 미확인 | private 서브모듈이라 코드 열람 불가 |
| LLM 토큰 비용 | 별도 발생 |

### 추천 전략: "두뇌는 코드, 몸통은 ToolJet"

```
Python/Node 로 에이전트 코어 작성 (LangGraph + Claude)
   ↓ REST API 로 노출
ToolJet REST API 플러그인으로 연결
   ↓
ToolJet 에서 UI + 인증 + 권한 + 승인 + 감사로그
```

에이전트 코드는 100줄, 나머지 "프로덕션화"는 ToolJet 이 담당한다.
여기에 **ToolJet MCP 서버**로 Claude Code 가 그 UI 까지 만들면 메타 에이전트 구성이 된다.

---

## 8. React / PHP로 만들 수 있는가

### 결론

**"클론"은 비현실적이고, "미니 버전"은 충분히 가능하다.**

### 규모 체크 (전수조사 결과)

```
server/src    TypeScript 1,205개 파일 (NestJS, 60+ 모듈)
frontend/src  JS/JSX     2,523개 파일 (React)
plugins       49 패키지
marketplace   45 패키지
+ EE 서브모듈 2개 (비공개)
+ Cypress E2E, Docusaurus 문서
= 수십 명 x 수년 (2021년부터 개발)
```

혼자서 전체 클론은 비현실적.

### React 로 만들기 — 가능 (프론트가 React 이므로 레퍼런스 완비)

"미니 로우코드 빌더" MVP 기준:

| 기능 | 라이브러리 | 참고할 ToolJet 코드 |
|---|---|---|
| 드래그앤드롭 캔버스 | `react-grid-layout`, `dnd-kit` | `frontend/src/AppBuilder/AppCanvas/` |
| 컴포넌트 레지스트리 | 직접 구현 | `WidgetManager/widgets/*.js` ← config 스키마를 그대로 참고 |
| 속성 패널 | `react-jsonschema-form` | `AppBuilder/` Inspector |
| `{{ }}` 표현식 | `jsep` + 안전한 evaluator | `frontend/AGENTS.md` resolution pipeline |
| 상태관리 | `zustand` | `AppBuilder/_stores/` (실제로 zustand 사용) |
| 쿼리 에디터 | `@monaco-editor/react` | `AppBuilder/QueryManager/` |
| 저장 포맷 | JSON | `app-import-export.service.ts` |

핵심 인사이트: `table.js` 같은 위젯 config 파일에 "properties / styles / events / defaults" 가 선언적 스키마로 들어있다. 이 패턴만 파악하면 미니 빌더 설계가 끝난다.

### PHP 로 만들기 — 백엔드만 가능

- 프론트는 PHP 로 불가 (캔버스/실시간 바인딩은 React/Vue 필요)
- 백엔드는 Laravel 로 충분히 포팅 가능

| ToolJet (NestJS) | Laravel 대응 |
|---|---|
| TypeORM | Eloquent |
| CASL ability guard | Policy / Gate |
| BullMQ 큐 | Laravel Queue (Redis) |
| 플러그인 시스템 | Service Provider + Contract |
| AES-256-GCM 암호화 | `Crypt` facade |
| WebSocket 멀티플레이어 | Laravel Reverb / Pusher |

추천 조합: **Laravel(API) + React(빌더) + PostgreSQL**

### 현실적 로드맵 (혼자, MVP)

| 단계 | 기간 | 내용 |
|---|---|---|
| 1 | 1주 | `table.js` 분석 → config 스키마 설계 |
| 2 | 2주 | React 캔버스 + 3개 컴포넌트 (Text/Button/Table) |
| 3 | 2주 | 속성 패널 + `{{ }}` 표현식 엔진 |
| 4 | 2주 | Laravel API + DB 쿼리 실행 |
| 5 | 2주 | 앱 저장/로드 + 뷰어 모드 |
| 6 | 2주 | 이벤트/액션 시스템 |
| **합계** | **~3개월** | 사용 가능한 미니 빌더 |

### 대안: ToolJet 을 "확장"하는 쪽이 투자수익이 높다

- 커넥터 플러그인 제작 → `@tooljet/cli` 로 1일
- 커스텀 컴포넌트 → `customComponent.js` 가 React 코드를 받아준다
- ToolJet 기반 제품화 (AGPL 주의)

단, 학습 목적이라면 미니 빌더를 직접 만들어보는 것을 권한다. 로우코드의 본질(스키마 기반 UI + 표현식 엔진)을 이해하는 가치가 크다.

---

## 9. 유튜브 강의 제작 가능성

### 결론: 가능하며, 지금이 적기

AGPL-3.0 은 소스코드 배포에 관한 라이선스이므로 강의 콘텐츠 제작·판매는 자유다.
(단, 로고 사용 시 상표권 주의. 수정 코드를 SaaS 로 서비스하면 공개 의무 발생.)

### 왜 지금인가

| 근거 | 내용 |
|---|---|
| 한국어 콘텐츠 희소 | ToolJet 한국어 강의가 거의 없다 → 선점 가능 |
| 검색 유입 | "사내 어드민", "Retool 대안", "로우코드" 꾸준한 수요 |
| AI 트렌드 결합 | "AI로 앱 만들기" 키워드 |
| 실무 수요 확실 | 모든 회사에 어드민이 필요 |
| 시연 임팩트 | 10분에 앱 완성 → 썸네일 소재 강력 |

### 추천 커리큘럼 (시즌제, 총 24편)

#### 시즌 1: 입문 (각 10~15분)

| EP | 제목 |
|---|---|
| 1 | "Retool 월 300만원? 이건 공짜" — ToolJet 소개 + Docker 5분 설치 |
| 2 | 첫 앱 만들기 — 10분에 고객관리 어드민 (조회수 기대) |
| 3 | 데이터소스 90개 완전정복 — PostgreSQL / MySQL / REST API |
| 4 | ToolJet DB — 엑셀 재고관리를 노코드 DB로 이전 |
| 5 | 이벤트/액션 — 코드 없이 CRUD 완성 |
| 6 | 배포 & 권한 — 팀에게 공개하고 권한 분리 |

#### 시즌 2: 실전 (각 20~30분)

| EP | 제목 |
|---|---|
| 7 | 실전1 주문관리 어드민 (환불/배송추적) |
| 8 | 실전2 매출 대시보드 (차트 + 필터 + Export) |
| 9 | 실전3 승인 워크플로우 |
| 10 | JS/Python 코드 심기 — 로우코드 한계 돌파 |
| 11 | 커스텀 컴포넌트 (React 코드 삽입) |
| 12 | Docker → AWS EC2 프로덕션 배포 (HTTPS, 백업) |

#### 시즌 3: AI 편 (최고 조회수 기대)

| EP | 제목 |
|---|---|
| 13 | **"Claude Code로 ToolJet 앱 만들기" — MCP 서버 연결 (킬러 영상)** |
| 14 | AI 챗봇 앱 — Anthropic 플러그인 연결 |
| 15 | RAG 구축 — Pinecone + 사내문서 검색 |
| 16 | AI 에이전트 — Workflow + Agent Node |
| 17 | Human-in-the-loop — AI 결과를 사람이 승인하는 UI |
| 18 | AI 에이전트 SI 로 수익화하기 (사업 편) |

#### 시즌 4: 개발자 심화 (충성도 높은 니치)

| EP | 제목 |
|---|---|
| 19 | 커넥터 플러그인 직접 만들기 — 토스페이먼츠 커넥터 라이브코딩 |
| 20 | 코드 투어1: NestJS 모듈 설계 + 상속 기반 에디션 분기 |
| 21 | 코드 투어2: 웹팩 NormalModuleReplacementPlugin 멀티에디션 구조 |
| 22 | 코드 투어3: CASL RBAC + 멀티테넌시 실전 |
| 23 | **"ToolJet 팀은 Claude Code를 이렇게 쓴다" — .claude/skills 16개 해부** |
| 24 | 미니 로우코드 빌더 직접 만들기 (React + Laravel) |

### 제작 팁

| 항목 | 추천 |
|---|---|
| 첫 10초 | 완성 화면부터 보여준다 ("이거 10분에 만들었습니다") |
| 썸네일 | "월 300만원 → 0원", "10분", "AI가 만들어줌" |
| 길이 | 입문 10~15분 / 실전 20~30분 |
| 구성 원칙 | 한 영상 = 한 결과물. 끝에 반드시 동작하는 산출물 |
| 깃허브 공개 | 영상별 완성 앱 JSON export 배포 → 구독 유도 |
| 쇼츠 분리 | "ToolJet 10초 팁" 시리즈로 유입 |

### 수익 경로

```
유튜브 광고 (조회수)
  ↓
인프런/클래스101 유료 강의 (심화 패키지, 10~20만원)
  ↓
기업 출강 / 사내교육 (회당 100~300만원)
  ↓
문의 유입 → 어드민 구축 외주 수주  ← 실제 수익의 핵심
  ↓
템플릿/플러그인 판매 (패시브)
```

유튜브는 수익 자체보다 **영업 깔때기**로 보는 것이 맞다.

---

## 10. 수익화 아이디어 완전판

### Tier S — 즉시 수익화 가능

#### 1) 사내 어드민 구축 외주 (SI)

| 항목 | 내용 |
|---|---|
| 단가 | 300만 ~ 1,500만원 / 건 |
| 공수 | 전통 개발 4주 → ToolJet 3~5일 (1/6 ~ 1/10) |
| 마진 | 70~85% |
| 타겟 | 중소기업, 스타트업, 병원, 학원, 물류, 쇼핑몰 |
| 난이도 | 낮음 |
| 초기투자 | 0원 (Docker 만 필요) |

마진이 큰 이유: 고객은 "어드민 개발비" 시장가(약 1,000만원) 기준으로 견적을 수락하는데, 실제 투입은 3~5일이다.

영업 멘트 예시:

> "보통 2개월 걸리는 어드민을 1주일에 납품합니다. 코드도 전부 드리고, 이후 수정은 **담당 직원이 직접 드래그로** 하실 수 있습니다."

마지막 문장(유지보수 독립)이 결정적인 설득 포인트다.

스타터 패키지 구성 예시:

| 플랜 | 금액 | 범위 |
|---|---|---|
| 베이직 | 300만 | 페이지 3개, 데이터소스 1개, 2주 납품 |
| 스탠다드 | 700만 | 페이지 8개, 데이터소스 3개, 권한 설정, 1개월 유지보수 |
| 프리미엄 | 1,500만 | 무제한 페이지, 워크플로우, AI 기능, 3개월 유지보수 + 교육 |

#### 2) 유지보수 / 호스팅 구독 (MRR 확보)

| 플랜 | 월 | 내용 |
|---|---|---|
| 베이직 | 20만 | 서버 모니터링, 업데이트, 백업 |
| 스탠다드 | 50만 | + 월 8시간 수정 작업 |
| 프리미엄 | 100만 | + 신규 앱 월 1개, 우선 대응 |

고객 10곳 x 50만 = 월 500만 패시브.
ToolJet 은 셀프호스팅이라 소형 VPS 기준 인프라 원가가 월 3~5만원 수준 → 마진 90%+.

주의: 코드 수정 없이 **배포/운영 서비스만** 제공하면 AGPL 안전. 코드를 수정해 SaaS 로 돌리면 공개 의무가 생긴다.

### Tier A — 레버리지가 큰 것

#### 3) 업종별 템플릿 판매

ToolJet 은 앱을 JSON 으로 export/import 할 수 있다 (`app-import-export.service.ts`) → 디지털 상품화에 최적.

| 템플릿 | 가격 | 타겟 |
|---|---|---|
| 병원 예약/환자 관리 | 50만 | 의원, 치과, 한의원 |
| 학원 수강생/출결 관리 | 40만 | 학원, 교습소 |
| 쇼핑몰 주문/재고 어드민 | 60만 | 스마트스토어 셀러 |
| 물류 배송 추적 | 60만 | 택배, 3PL |
| CRM (영업 파이프라인) | 50만 | B2B 스타트업 |
| 매출 대시보드 팩 | 30만 | 전 업종 |
| 생산/설비 모니터링 | 80만 | 제조업 |

경제성: 템플릿 1개 제작 3일 → 10곳 판매 500만원. 100곳에 팔아도 추가 원가 0원.

판매 채널: 자체 랜딩페이지, 크몽, 노션 마켓, Gumroad, 유튜브 설명란.

#### 4) 한국형 커넥터 플러그인 (선점 가치)

ToolJet 마켓플레이스에 **한국 서비스 커넥터가 전무**하다.

| 커넥터 | 수요 |
|---|---|
| 토스페이먼츠 / 포트원 | 모든 결제 어드민 |
| 카카오 알림톡 / 비즈메시지 | 모든 CS 어드민 |
| 네이버 커머스 API | 스마트스토어 셀러 전체 |
| 더존 / 이카운트 ERP | 중소기업 회계 |
| CJ대한통운 / 스윗트래커 | 물류 |
| 공공데이터포털 | 공공 프로젝트 |
| 솔라피 / NHN Toast SMS | 알림 |

제작 방법:

```bash
npx @tooljet/cli plugin create tosspayments
# plugins/_templates 참고 → manifest.json + lib/index.ts 작성
# marketplace/plugins/anthropic/ 구조를 레퍼런스로 활용
```

수익 모델:

- 무료 공개 → 마켓플레이스 노출 → **인바운드 외주 문의** (가장 추천)
- 유료 라이선스 (건당 30~100만)
- 기업 커스텀 커넥터 개발 (건당 200~500만)

#### 5) AI 에이전트 SI (최고 단가)

| 상품 | 단가 | 구성 |
|---|---|---|
| 사내 AI 문서검색 챗봇 | 800~2,000만 | Pinecone RAG + Chat UI + 권한 |
| CS 자동응답 + 사람 승인 | 1,000~2,500만 | LLM + Human-in-the-loop UI |
| 서류 자동처리 | 1,500~3,000만 | Textract + LLM + 검증 UI |
| AI 데이터 분석 어드민 | 1,000~2,000만 | 자연어 → SQL → 차트 |

경쟁 우위: 경쟁사는 에이전트 로직만 납품하고 "UI 는 별도 견적"이라 하지만, ToolJet 을 쓰면 **UI + 인증 + 권한 + 승인플로우 + 감사로그**를 함께 납품할 수 있다. 특히 감사로그는 금융/의료/공공에서 필수 요건이다.

기술 스택:

```
[두뇌] Python/Node + Claude (LangGraph) → REST API
[몸통] ToolJet (REST API 플러그인 연결)
       + Chat 컴포넌트, 승인 버튼, 권한, 감사로그
[배포] Docker + K8s (deploy/ 폴더 활용)
```

### Tier B — 브랜드 / 깔때기

#### 6) 교육 사업

| 상품 | 가격 | 비고 |
|---|---|---|
| 유튜브 | 광고수익 | 영업 깔때기 |
| 인프런/클래스101 강의 | 10~20만 | 패시브 |
| 기업 사내교육 | 회당 100~300만 | 마진 최고 |
| 1:1 컨설팅 | 시간당 15~30만 | |
| 부트캠프 (4주) | 1인 80만 x 15명 = 1,200만/기 | |

"노코드 + AI 사내앱 개발자" 과정은 비개발자 직장인 타겟 수요가 크다.

#### 7) 브랜드 포지셔닝

```
브랜드: "3일 만에 어드민, AI까지 붙여서"
채널: 유튜브 + 블로그 + 링크드인
무료 미끼: 템플릿 1개 무료 배포, 커넥터 오픈소스 공개
유입 → 상담 → 외주 → 유지보수 구독
```

깔때기 수치 예시:

```
유튜브 월 5만뷰 → 상담 20건 → 수주 4건 x 700만 = 2,800만/월
                            → 유지보수 4건 x 50만 = +200만 MRR 누적
```

### 종합 비교표

| 아이디어 | 수익 | 난이도 | 회수기간 | 확장성 | 추천도 |
|---|---|---|---|---|---|
| 1) 어드민 외주 | 최상 | 낮음 | 1개월 | 낮음 | 즉시 시작 |
| 2) 유지보수 구독 | 상 | 매우 낮음 | 즉시 | 상 | 1)과 묶기 |
| 3) 템플릿 판매 | 중상 | 낮음 | 2개월 | 최상 | 권장 |
| 4) 커넥터 플러그인 | 중상 | 중 | 3개월 | 상 | 선점 가치 |
| 5) AI 에이전트 SI | 최상 | 상 | 3개월 | 중 | 최고 단가 |
| 6) 교육/강의 | 중 | 중 | 6개월 | 최상 | 깔때기 |
| 7) 브랜딩 | 간접 | 중 | 6개월 | 최상 | 장기 |

---

## 11. 90일 실행 플랜

```
Week 1-2   Docker 설치 → 샘플 어드민 3개 제작
           (주문관리 / 매출대시보드 / 재고관리)
Week 3-4   포트폴리오 랜딩페이지 + 유튜브 EP1~3 업로드
Week 5-6   크몽/숨고 등록 → 첫 외주 수주 (레퍼런스용 저가 1건)
Week 7-8   토스페이먼츠 또는 카카오 알림톡 커넥터 제작 → 오픈소스 공개
           → 마켓플레이스 PR → 유튜브 EP19
Week 9-10  업종 템플릿 2개 완성 → 판매 시작
Week 11-12 ToolJet MCP + Claude 조합 영상 (EP13) → AI SI 상담 접수
───────────────────────────────────────────────
90일 목표: 외주 2건 + 유지보수 2건 + 템플릿 판매 개시
          = 월 매출 약 1,500만 + MRR 100만
```

---

## 12. 리스크 체크리스트

| 리스크 | 대응 |
|---|---|
| AGPL 전염성 | 코드 수정 SaaS 금지 / 외주·운영서비스는 안전 |
| Workflows·AI 가 EE 유료 | AI 는 외부 API 로 직접 연결하면 CE 에서도 구현 가능 |
| 현재 버전이 beta (3.21.71-beta) | 프로덕션은 LTS 태그 사용 (`ee-lts-latest`) |
| ToolJet 정책 변경 | 고객 데이터·앱 JSON 은 항상 export 백업 |
| 고객이 "직접 하겠다" | 유지보수 구독을 처음부터 끼워 판매 |
| `LOCKBOX_MASTER_KEY` 유실 | 안전한 시크릿 저장소에 백업, `rotate:keys` 숙지 |
| EE 서브모듈 접근 불가 | 공개 저장소 증거만으로 분석/이슈 작성 (public/private 경계 준수) |
| Node 버전 불일치 | 브랜치마다 다를 수 있으므로 항상 `nvm use` |

---

## 13. 참고 링크

| 항목 | 링크 |
|---|---|
| **이 분석 대상 저장소 (내 포크)** | **https://github.com/bmshin94/ToolJet** |
| 업스트림 원본 저장소 | https://github.com/ToolJet/ToolJet |
| EE 서버 서브모듈 (비공개) | https://github.com/ToolJet/ee-server |
| EE 프론트 서브모듈 (비공개) | https://github.com/ToolJet/ee-frontend |
| 공식 문서 | https://docs.tooljet.com |
| 설치 가이드 | https://docs.tooljet.com/docs/setup/ |
| **MCP 가이드 (베타)** | **https://docs.tooljet.com/docs/build-with-ai/mcp/overview** |
| 컴포넌트 레퍼런스 | https://docs.tooljet.com/docs/widgets/button |
| 데이터소스 레퍼런스 | https://docs.tooljet.com/docs/data-sources/airtable/ |
| ToolJet Cloud | https://tooljet.com |
| `@tooljet/cli` (npm) | https://www.npmjs.com/package/@tooljet/cli |
| 이슈 트래커 | https://github.com/ToolJet/ToolJet/issues |
| 로드맵 | https://github.com/orgs/ToolJet/projects/15 |
| Slack 커뮤니티 | https://tooljet.com/slack |
| X (Twitter) | https://twitter.com/ToolJet |
| Model Context Protocol | https://modelcontextprotocol.io/introduction |
| AWS Marketplace | https://aws.amazon.com/marketplace/pp/prodview-fxjto27jkpqfg |
| Azure Marketplace | https://azuremarketplace.microsoft.com/en-us/marketplace/apps/tooljetsolutioninc1679496832216.tooljet |

### 저장소 내부 참고 파일

| 파일 | 내용 |
|---|---|
| `AGENTS.md` / `CLAUDE.md` | 저장소 전체 아키텍처 가이드 |
| `.agents/context/product-map.md` | 제품 지도 |
| `.agents/context/architecture-map.md` | 시스템 지도 |
| `UBIQUITOUS_LANGUAGE.md` | 도메인 용어 사전 |
| `frontend/AGENTS.md` | 프론트 규약 + 앱빌더 아키텍처 |
| `server/AGENTS.md` | 백엔드 + 테스트 규약 |
| `server/docs/testing.md` | 백엔드 테스트 가이드 |
| `CONTRIBUTING.md` | 기여 가이드 |
| `plugins/README.md` | 플러그인 제작 가이드 |
| `.env.example` | 환경변수 전체 목록 |
| `docker-compose.yaml` | 개발환경 6개 컨테이너 구성 |
| `deploy/` | K8s / Helm / EC2 / OpenShift 배포 자산 |

---

## 부록: 핵심 요약 (한 장)

- **ToolJet 은** 사내 업무앱을 드래그앤드롭 + AI 로 만드는 **오픈소스 로우코드 플랫폼** (Retool 대안)
- **구성은** React 앱빌더(83 컴포넌트) + NestJS 백엔드(60+ 모듈) + 커넥터 94개 + PostgreSQL/Redis
- **플러그인도 스킬도 MCP 도 아니고**, 그 셋을 모두 **제공하는 제품**이다
- **토큰은** 브라우저 사용 시 불필요, CLI/MCP/자동화 시 **PAT(`tj_pat_`)** 필요 (워크스페이스 바인딩 + 필수 만료)
- **AI 에이전트의 "몸통"**(UI·인증·권한·툴·승인·감사로그)으로 매우 유용. 두뇌는 코드로 짜고 ToolJet 에 연결
- **React 로 미니 버전 3개월 가능**, PHP 는 Laravel 백엔드만, 전체 클론은 비현실적
- **유튜브 강의 가능** (AGPL 제약 없음). 한국어 콘텐츠 선점 기회
- **수익화 1순위는 어드민 외주(마진 70%+) + 유지보수 구독(MRR)**, 레버리지는 템플릿·커넥터, 최고단가는 AI 에이전트 SI
- **라이선스 주의**: 코드 수정해서 SaaS 로 서비스하면 소스 공개 의무 (AGPL-3.0)

---

*이 문서는 `https://github.com/bmshin94/ToolJet` 저장소를 전수조사하여 Claude Code 가 작성했습니다.*
