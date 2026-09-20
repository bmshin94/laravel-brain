# Laravel Brain 전수조사 & 활용 전략 정리 🧠

> 카리나(Claude Code)와 함께 진행한 `laravel-brain` 저장소 전수조사 대화 정리본
> 작성일: 2026-09-20

---

## 🔗 관련 링크

| 구분 | 주소 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/laravel-brain |
| **원본 저장소 (upstream)** | https://github.com/laramint/laravel-brain |
| 공식 랜딩 페이지 | https://laramint.dev/ |
| 공식 문서 사이트 | https://laravel-brain.laramint.dev/ |
| Releases | https://github.com/laramint/laravel-brain/releases |
| OSS Insight 통계 | https://ossinsight.io/analyze/laramint/laravel-brain |
| Packagist | https://packagist.org/packages/laramint/laravel-brain |
| 연관 패키지 (스트레스 테스트) | https://github.com/LaraMint/laravel-stress |
| 연관 패키지 (보안 스캐너) | `laramint/laravel-security-scanner` |
| Model Context Protocol | https://modelcontextprotocol.io |
| Laravel MCP 패키지 | https://github.com/laravel/mcp |
| laravel/ai 패키지 | https://github.com/laravel/ai |

---

## 1. 이게 뭐 하는 물건인가

### 📌 정체

**`laramint/laravel-brain`** — Laravel 전용 개발 도구 Composer 패키지.
내 라라벨 프로젝트를 통째로 정적 분석해서 **"요청 → 컨트롤러 → 서비스 → 모델 → 잡/이벤트"** 흐름 전체를
**인터랙티브 노드 그래프**로 렌더링해준다.

| 항목 | 내용 |
|---|---|
| 패키지명 | `laramint/laravel-brain` |
| 라이선스 | MIT (상업적 이용 가능) |
| 요구사항 | PHP 8.0+, Laravel 9 / 10 / 11 / 12 / 13 |
| 백엔드 규모 | `src/` PHP **120개 파일 / 34,575줄** |
| 테스트 | `tests/` **427개 파일** (Pest) |
| 프론트엔드 | React 19 + TypeScript + Vite + D3 + Dagre |
| 문서 | VitePress (`docs/`) |
| 주요 기여자 | MrMarchOne(Abdelrahman Muhammed), webard, Sander Muller 등 |
| 후원 | GitHub Sponsors(`mrmarchone`), Buy Me a Coffee |

### 📂 폴더 구조

```
laravel-brain/
├── src/                          ← 백엔드 핵심 (PHP 120파일)
│   ├── Analysis/    (80+ 파일)   ← 심장부: 정적 분석기 모음
│   │   ├── RouteAnalyzer         라우트 수집
│   │   ├── ControllerAnalyzer    컨트롤러 클래스/메서드 해석
│   │   ├── MethodTracer          호출 체인 깊이 추적 (핵심)
│   │   ├── ModelAnalyzer         Eloquent 관계 추출
│   │   ├── QueryTracer           메서드별 DB 쿼리 탐지
│   │   ├── SecurityAnalyzer      보안 취약점 탐지
│   │   ├── FilamentAnalyzer      Filament 패널/리소스 탐색
│   │   ├── AiAnalyzer            laravel/ai 에이전트 탐지
│   │   ├── GitChurnAnalyzer      git 커밋 빈도 분석
│   │   ├── Reachability/         미도달 클래스 역추적
│   │   └── Incremental/          변경분만 재스캔
│   ├── Parser/                   nikic/php-parser 기반 AST 파싱
│   ├── Graph/                    Node · Edge · Graph + GraphBuilder
│   ├── Ai/                       AI 컨텍스트 / 룰 내보내기
│   ├── Mcp/                      MCP 서버 (툴 10 + 리소스 2)
│   ├── Commands/                 artisan 커맨드 3종
│   ├── Http/                     뷰어 컨트롤러 + 동일출처 미들웨어
│   └── Storage/                  file / database 저장 드라이버
├── frontend/                     React SPA 그래프 뷰어
├── config/laravel-brain.php      설정 파일 (44KB, 주석 상세)
├── routes/brain.php              /_laravel-brain 라우트
├── resources/                    빌드된 뷰어 (블레이드 + 에셋)
├── database/migrations/          그래프 저장 테이블
├── benchmark/                    성능 벤치마크
├── docs/                         VitePress 문서
└── tests/                        Pest 테스트 427개
```

### ⚙️ 동작 파이프라인

```
php artisan brain:scan
        │
        ├─ ① PHP 파일을 AST(추상 구문 트리)로 파싱  ※ 실행 X, 읽기만 O
        ├─ ② 80여 개 Analyzer가 정보 수집
        │     라우트/미들웨어/컨트롤러/서비스/모델/잡/이벤트/리스너/
        │     옵저버/폴리시/스케줄/브로드캐스트/Filament/매크로/캐시/HTTP
        ├─ ③ GraphBuilder가 노드 + 엣지로 조립 (뚱뚱한 클래스·복잡도 핫스팟 표시)
        └─ ④ JSON 저장 → storage/app/laravel-brain/.graph-*.json

http://localhost:8000/_laravel-brain
        └─ React SPA가 JSON을 읽어 인터랙티브 그래프로 렌더링
```

### 🎯 언제 쓰나

1. **레거시 프로젝트 인수** — 코드 안 읽고 전체 구조 파악 (최대 용도)
2. **영향 범위 추적** — "이 모델 건드리면 어디가 터지나"
3. **리팩토링 사전 조사** — 복잡도 핫스팟, 300줄 초과 뚱뚱한 클래스 색출
4. **버그 예측** — Riskiest Files: *커밋 빈도 × 복잡도* ("code as a crime scene" 기법)
5. **죽은 코드 후보 탐색** — Reachability 탭
6. **보안 점검** — 인증 없이 공개된 라우트 확인
7. **온보딩 자료** — 아키텍처 다이어그램 자동 생성 (PNG / Mermaid)
8. **AI 컨텍스트 생성** — 아래 3가지 기능이 핵심

### 💖 나에게 주는 가치

- **AI 컨텍스트 자동 생성** — 노드 클릭 → 🤖 버튼 → 토큰 최적화 마크다운 클립보드 복사
  (호출 체인 · DB 작업 · 캐시 작업 · 소스 스니펫 · 패키지 목록, 기본 6,000 토큰 예산, 결정론적 출력)
- **AI 룰 파일 8종 자동 생성** — 실제 코드 구조 기반으로 `CLAUDE.md` 등을 생성
- **MCP 서버 내장** — Claude Code가 그래프에 직접 질의 가능

---

## 2. 쉽게 이해하는 비유

### 🏙️ 비유 1 — 프로젝트는 지도 없는 도시

| 도시 | 라라벨 |
|---|---|
| 출입구 / 관문 | 라우트 (`GET /users`) |
| 검문소 | 미들웨어 (`auth`) |
| 구청 창구 | 컨트롤러 |
| 실무 부서 | 서비스 클래스 |
| 서류 보관소 | 모델 (DB) |
| 대기 업무함 | 잡(Job) |
| 방송 → 부서 반응 | 이벤트 / 리스너 |
| 새벽 환경미화차 | 스케줄 |

→ **Laravel Brain = 이 도시의 지하철 노선도를 자동으로 그려주는 기계.**

### 🏥 비유 2 — 코드 건강검진 MRI

```
brain:scan = 전신 MRI 촬영
     ↓ 결과지
🔴 "이 컨트롤러 비만입니다"        (300줄 초과 / 메서드 10개 초과)
🟠 "이 메서드 혈관이 꼬였습니다"   (순환복잡도 Critical)
🟡 "이 라우트는 문이 열려있습니다" (인증 미들웨어 없음)
⚪ "이 클래스 17개는 아무도 안 씁니다" (Reachability)
💥 "이 파일이 다음에 터질 확률 1위"  (커밋 빈도 × 복잡도)
```

### 🔬 정적 분석 vs 동적 분석

| 구분 | 비유 | 대표 도구 |
|---|---|---|
| **동적 분석** | 자동차를 실제로 몰아보며 문제 찾기 | Telescope, Debugbar |
| **정적 분석** | 자동차를 세워놓고 설계도를 읽어 문제 찾기 | **Laravel Brain** |

→ 서버 안 켜도 되고, DB 데이터 없어도 되고, 요청 한 번 안 보내도 된다.

### 🎨 비유 3 — 결과 화면은 구글 맵

```
[왼쪽 사이드바]     [가운데 캔버스]            [오른쪽 사이드바]
 탭 목록             노드 + 엣지 그래프         선택 노드 상세
 - 라우트별 탭       휠 줌 / 드래그 팬          - Info (DB/캐시/HTTP)
 - 커맨드 / 스케줄   🟢라우트 🔵컨트롤러         - Source (실제 코드)
 - Riskiest Files    🟣서비스 🔴모델 🟡이벤트     - Flow (순서도)
 - AI Agents                                    - Sequence Diagram
 - Reachability                                 - Git 히스토리 + Diff
                                                - 🤖 AI 컨텍스트 복사
```

---

## 3. 핵심 Q&A

### Q1. 설치 및 사용법

```bash
# 설치 (반드시 --dev! 개발 도구이며 local 환경에서만 라우트/커맨드 등록됨)
composer require --dev laramint/laravel-brain

# 기본 3단계
php artisan brain:scan
php artisan serve
# → http://localhost:8000/_laravel-brain
```

**스캔 옵션**

```bash
php artisan brain:scan --memory-limit=2048M   # 기본 1024M, 최소값도 1024M
php artisan brain:scan --memory-limit=-1      # 무제한 (주의)
php artisan brain:scan --watch                # 파일 변경 시 자동 재스캔
php artisan brain:scan --watch --interval=5   # 폴링 주기 (기본 3초)
php artisan brain:scan --auto-discover        # 실제 라우터에서 라우트 수집
```

**AI 관련 커맨드**

```bash
php artisan brain:export-context
php artisan brain:export-context --route="GET /users" --budget=4000
php artisan brain:export-context --node="action::App\Http\Controllers\UserController::index"
php artisan brain:export-context --output=/tmp/context.md
php artisan brain:export-context --format=json

php artisan brain:generate-rules                  # 8종 전부
php artisan brain:generate-rules --target=claude  # 특정 대상만
php artisan brain:generate-rules --dry-run
php artisan brain:generate-rules --force
```

**생성되는 AI 룰 파일 8종**

| 대상 | 파일 |
|---|---|
| `claude` | `CLAUDE.md` |
| `cursor` | `.cursor/rules/laravel-brain.mdc` |
| `windsurf` | `.windsurf/rules/laravel-brain.md` |
| `copilot` | `.github/copilot-instructions.md` |
| `junie` | `.junie/guidelines.md` |
| `aider` | `CONVENTIONS.md` |
| `agents` | `AGENTS.md` |
| `codex` | `CODEX.md` |

**설정 퍼블리시 & 주요 키**

```bash
php artisan vendor:publish --tag=laravel-brain-config
```

```php
'driver'        => env('LARAVEL_BRAIN_DRIVER', 'file'),   // file | database
'memory_limit'  => env('LARAVEL_BRAIN_MEMORY_LIMIT', '1024M'),
'source_paths'  => ['app', 'src'],
'watch_paths'   => ['app', 'routes', 'config'],
'auto_discover_routes' => false,
'reachability'  => ['enabled' => false],   // 무거워서 기본 off
'ai'            => ['enabled' => true],
'actions'       => ['paths' => ['app/Actions']],
'mcp'           => ['enabled' => true],
```

**출력 파일** — `storage/app/laravel-brain/.graph-manifest.json`, `.graph-{tab-id}.json`
→ `.gitignore`에 `storage/app/laravel-brain/` 추가 권장.

**DB 드라이버** (storage/ 사용 불가 환경)

```dotenv
LARAVEL_BRAIN_DRIVER=database
LARAVEL_BRAIN_DB_TABLE=laravel_brain_graphs
```
테이블은 첫 스캔 시 자동 생성되므로 마이그레이션 불필요.

---

### Q2. 플러그인? 스킬? MCP?

**정답: Composer(Laravel) 패키지이며, MCP 서버를 옵션으로 내장.**

| 분류 | 해당 | 비고 |
|---|:---:|---|
| Composer 패키지 | ✅ | 본질 |
| Laravel 패키지 | ✅ | `LaravelBrainServiceProvider` auto-discovery |
| 웹 앱 | ✅ | `/_laravel-brain`에 React SPA 서빙 |
| CLI 도구 | ✅ | artisan 커맨드 3종 |
| MCP 서버 | 🔶 | 옵션 — `laravel/mcp` 설치 시에만 활성화 |
| Claude Skill | ❌ | 아님 |
| Claude Code 플러그인 | ❌ | 아님 |
| VSCode 확장 | ❌ | 아님 |

**MCP 활성화**

```bash
composer require --dev laravel/mcp        # Laravel 11+ 필요
claude mcp add brain -- php artisan mcp:start brain
```

> ⚠️ `laravel/mcp`는 `symfony/process ^7.4.5|^8.0.5`를 요구 → Laravel 9/10의 `symfony/process ^6.x`와 충돌.
> PHP 버전이 아니라 **의존성 충돌**이 제약이다. `LARAVEL_BRAIN_MCP_ENABLED=false`로 끌 수 있음.

**MCP 툴 10종**

| 툴 | 역할 |
|---|---|
| `brain_get_manifest` | 마지막 스캔의 탭 인덱스 + 위험도 |
| `brain_get_context` | 라우트/노드 포커스 컨텍스트 |
| `brain_find_usages` | 노드의 모든 직접 호출자 (파일별 그룹) |
| `brain_get_route_security` | 라우트별 노출도/위험도 |
| `brain_get_subgraph` | 특정 탭의 노드 + 엣지 |
| `brain_get_graph` | 전체 병합 그래프 (타입 필터) |
| `brain_get_agent_rules` | 룰 파일 내용을 파일 쓰지 않고 조회 |
| `brain_get_file_history` | 마지막 커밋자 + diff |
| `brain_get_riskiest_files` | 위험 파일 랭킹 |
| `brain_rescan` | 재스캔 후 저장 |

+ 리소스 2종: `ManifestResource`, `SubgraphResource`

---

### Q3. API 토큰이 필요한가? → **아니오, 전혀 필요 없다.**

| 기능 | 토큰 | 이유 |
|---|:---:|---|
| 코드 스캔 | ❌ | 100% 로컬 AST 정적 분석 |
| 그래프 뷰어 | ❌ | 내 라라벨 서버가 서빙 |
| AI 컨텍스트 내보내기 | ❌ | **생성만** 함 (붙여넣기는 사용자가) |
| AI 룰 파일 생성 | ❌ | 로컬 파일 쓰기 |
| MCP 서버 | ❌ | 로컬 stdio 프로세스 |
| Git 히스토리 / Diff | ❌ | 로컬 `.git` 읽기 |
| DB 스키마 / 통계 | ❌ | 기존 DB 연결 사용 |
| 보안 스캔 | ❌ | 로컬 룰 기반 |
| 스트레스 테스트 | ❌ | 로컬 URL 화이트리스트 |

**오해하기 쉬운 3가지**

1. `laravel/ai` 에이전트 트레이싱 → 코드 안의 에이전트 클래스를 **발견**만 함. LLM 호출 안 함.
2. AI 컨텍스트 내보내기 → Claude API 호출이 아니라 **붙여넣을 텍스트 생성**.
3. MCP 서버 → Claude Code가 이미 인증돼 있어 별도 토큰 불필요.

**보안 설계 포인트**

- `/_laravel-brain` 라우트는 `$this->app->isLocal()` 가드로 **local 환경에서만** 등록
- `--no-dev` 배포 시 프로덕션엔 아예 부재
- POST 엔드포인트(scan / stress-test / generate-rules)는 `EnsureRequestIsSameOrigin` 미들웨어 보호
- 스트레스 테스트 대상은 서버에서 화이트리스트 검증 (SSRF 방어):
  `localhost`, `127.0.0.1`, `*.test`, `*.local`, `*.ddev.site`, 단일 라벨 도커 서비스명,
  사설 IPv4 대역(10.x / 172.16–31.x / 192.168.x), `APP_URL` 호스트

---

### Q4. 왜 깃허브에서 유명한가

원본 저장소 기준 **⭐ 약 785개** (조회 시점에 따라 변동).

1. **진짜 아픈 곳을 긁음** — Telescope(런타임), Debugbar(요청)는 있었지만 **"아키텍처 전체 지도"** 도구는 공백이었다.
2. **Zero-config** — 설치 후 명령어 하나로 작동. 진입 장벽 0.
3. **스크린샷이 예쁨** — 시각적 도구는 X / Reddit / Laravel News 바이럴 계수가 높다.
4. **AI 시대 타이밍** — 컨텍스트 내보내기 + 룰 8종 + MCP 서버 = "AI 에이전트용 코드베이스 인덱서" 포지셔닝.
5. **기능 밀도** — README 47KB. Filament, 매크로, 트랜잭션 경계, Job 체인/배치, 브로드캐스트, 옵저버,
   폴리시, 지연 프로바이더 진단, Blade 뷰 구성, API 리소스, 시퀀스 다이어그램, Git Diff, 스트레스 테스트…
6. **엔지니어링 퀄리티** — 테스트 427개, PHPStan(larastan) + Pint + husky pre-commit,
   GitHub Actions 4종(ci / benchmark / benchmark-comment / pr-template), 벤치마크 스위트, 증분 스캔.
7. **문서의 지적 정직함** — "Reachability는 죽은 코드 보고서가 **아니다**", "`#[UseSmartestModel]`은
   티어지 모델명이 아니므로 추측하지 않는다", Reachability 기본 off 이유를 실측치(최악 ×3.0 / 최선 +2%)로 설명.
8. **라라벨 생태계 문화** — 고품질 패키지 → Laravel News → SNS 확산 경로가 잘 작동한다.

---

### Q5. 로컬 에이전트 구축에 도움이 되는가 → **매우 그렇다.**

**해결하는 문제**

```
❌ 기존: "UserController 수정해줘" → 호출 관계를 모름 → 전체 코드 읽으면 토큰 폭발
✅ Brain: 그래프에서 호출 체인 · DB · 캐시 · 복잡도 · 스니펫을
          6,000 토큰짜리 구조화 컨텍스트로 제공
```

**활용 시나리오**

- **A. MCP 직결 (최강)** — `claude mcp add brain -- php artisan mcp:start brain`
  → "인증 없이 뚫린 라우트 있어?" → `brain_get_route_security`
  → "OrderService 고치면 뭐가 영향받아?" → `brain_find_usages`
- **B. 룰 파일 상시 주입** — `brain:generate-rules --target=claude`
  ※ 현재 `CLAUDE.md`에는 카리나 페르소나가 있으므로 `--target=agents`로 `AGENTS.md`에 넣거나 섹션 분리 권장
- **C. 파이프라인 연동** — `brain:export-context --format=json | python my_agent.py`
  또는 `GET /_laravel-brain/api/context` HTTP 엔드포인트

> ⚠️ **주의**: 모든 MCP 툴은 *마지막에 저장된 스캔*을 읽는다(라이브 파일시스템 X).
> 코드 변경 후엔 `brain_rescan`을 먼저 호출하거나 `--watch`를 병행할 것.

**직접 만들 만한 에이전트**

1. 코드 리뷰 봇 — 변경 파일의 Riskiest 점수 + 복잡도로 리뷰 포인트 제시
2. 온보딩 봇 — "주문 기능 어떻게 돌아가요?" → 그래프 체인으로 설명
3. 영향도 분석 봇 — PR 오픈 시 영향 범위 자동 코멘트
4. 보안 감시 봇 — 새 라우트에 인증 미들웨어 누락 시 경고
5. 리팩토링 우선순위 봇 — 주간 Riskiest Files 리포트 슬랙 전송

---

### Q6. React나 PHP로 만들 수 있는가 → **이미 React + PHP로 만들어져 있다.**

**백엔드 (PHP)**

```json
"require": {
  "php": ">=8.0",
  "nikic/php-parser": "^5.0",
  "illuminate/console": "^9|^10|^11|^12|^13",
  "illuminate/support": "^9|^10|^11|^12|^13",
  "laramint/laravel-stress": "^1.0"
}
```

**프론트엔드 (React 19 + TS)**

```json
"react": "^19.2.5", "d3": "^7.9.0", "dagre": "^0.8.5",
"diff": "^5.2.2", "html2canvas": "^1.4.1",
"react-syntax-highlighter": "^16.1.1", "@floating-ui/react": "^0.27.19",
"vite": "^8.0.10", "typescript": "~6.0.2"
```

**빌드 트릭**

```json
"build": "tsc -b && vite build && mv ../resources/assets/index.html ../resources/views/index.blade.php"
```
→ Vite 산출물 `index.html`을 블레이드 뷰로 rename. 별도 서버 프로세스가 필요 없는 이유.

**직접 만들 때 난이도**

| 파트 | 난이도 |
|---|:---:|
| PHP AST 파싱 기초 (`nikic/php-parser`) | ⭐⭐ |
| 라우트 / 컨트롤러 추출 | ⭐⭐ |
| React 그래프 렌더링 (D3 / dagre) | ⭐⭐⭐ |
| **호출 체인 추적** | ⭐⭐⭐⭐⭐ |
| DI 컨테이너 바인딩 해석 (인터페이스→구현체) | ⭐⭐⭐⭐⭐ |
| Facade accessor 해석 | ⭐⭐⭐⭐ |
| 증분 스캔 (핑거프린트 + 머지) | ⭐⭐⭐⭐ |
| 대규모 앱 메모리 관리 | ⭐⭐⭐⭐ |

**결론:** 처음부터 만들기보다 **포크해서 기능을 얹는 쪽**이 훨씬 효율적.
실제로 이 포크에는 이미 로컬 커밋이 존재한다.

- `feat(inspector): add Service Explorer & Method Logic Inspector`
- `fix(scan): make the web-triggered scan's time and memory limits configurable`
- `docs: created CLAUDE.md persona guide`

**새로 만든다면 추천 방향**
- 🥇 다른 프레임워크용 (`symfony-brain`, `nest-brain`, `django-brain`) — 니치 시장이 비어있음
- 🥈 특화 경량 도구 (예: Eloquent 관계만 ERD로)
- 🥉 SaaS 래퍼 (CI에서 스캔 결과 수집 → 히스토리/트렌드 대시보드)

---

## 4. 수익화 아이디어

> ⚖️ MIT 라이선스 — 상업적 이용/수정/재배포/유료 판매 모두 가능. 저작권 고지 + 라이선스 사본 유지 필요.
> 단, 브랜드명(`Laravel Brain`, `LaraMint`)을 그대로 쓰는 건 오해 소지가 있으므로 별도 네이밍 권장.
> `Laravel`은 Taylor Otwell의 상표이므로 제품명 앞에 붙이는 것도 피하는 편이 안전하다.

### 💎 1. SaaS — 코드베이스 헬스 대시보드
**수익성 💰💰💰💰💰 / 난이도 상**

로컬 도구는 "한 번 보고 끝". **시간에 따른 추세**는 아무도 안 보여준다.

```
CI에서: php artisan brain:scan --format=json | curl -X POST https://our-saas.com/ingest
대시보드에서:
  📈 복잡도 전월 대비 +12%
  📉 미도달 클래스 45 → 62개
  🔥 OrderService.php 4주 연속 Riskiest 1위
  🚨 새 라우트 3개가 auth 미들웨어 없이 추가됨
  📊 팀원별 / 스프린트별 기술부채 추이 + 슬랙 알림
```

| 플랜 | 가격 | 내용 |
|---|---|---|
| Free | $0 | 1 레포, 30일 히스토리 |
| Pro | $29/월 | 5 레포, 무제한 히스토리, 슬랙 알림 |
| Team | $99/월 | 무제한 레포, PR 코멘트, SSO |
| Enterprise | $500+/월 | 온프레미스, 커스텀 룰, SLA |

핵심: 개발자 개인은 도구에 돈을 안 써도, **CTO/팀리드는 "기술부채 가시화"에 돈을 쓴다.**
MVP = 인제스트 API + 시계열 차트. 2~3개월.

### 💎 2. 레거시 라라벨 감사(Audit) 서비스 ⭐ 최우선 추천
**수익성 💰💰💰💰💰 / 난이도 하**

도구를 파는 게 아니라 **도구로 서비스를 판다.**

```
고객: "5년 된 라라벨 프로젝트를 인수받았는데 막막해요"
  ↓ brain:scan + 분석 + 리포트 작성
납품: 감사 리포트 (아키텍처 다이어그램 / 보안 취약 라우트 /
      기술부채 핫스팟 Top 20 / 죽은 코드 후보 /
      리팩토링 우선순위 로드맵 3·6·12개월 / 예상 공수)
가격: 건당 300~1,500만원
```

- 초기 투자 **0원**, 오늘 당장 시작 가능
- 템플릿을 만들어두면 2회차부터 반나절
- **감사 → 리팩토링 계약**으로 자연스럽게 연결 (진짜 큰 돈)
- 고객 채널: 크몽 / 위시켓 / 원티드 프리랜서 / 링크드인 / 라라벨 커뮤니티

### 💎 3. 유료 프로 애드온 (Open Core)
**수익성 💰💰💰💰 / 난이도 중**

```bash
composer require laramint/laravel-brain    # 무료
composer require ourbrand/brain-pro --dev  # 유료 애드온
```

| 프로 기능 | 지불 동기 |
|---|---|
| 팀 공유 스냅샷 | 그래프를 URL로 공유 (온보딩 자료) |
| PR 영향도 자동 코멘트 | GitHub Action |
| 커스텀 아키텍처 룰 | "컨트롤러는 모델 직접 호출 금지" 위반 시 CI 실패 |
| Confluence/Notion 내보내기 | 아키텍처 문서 자동 동기화 |
| 멀티 프로젝트 비교 | 마이크로서비스 통합 뷰 |
| PDF 리포트 생성기 | 경영진 보고용 |

가격: 개발자당 연 $99 / 팀 연 $499 (Anystack, Lemon Squeezy 등으로 라이선스 관리)

### 💎 4. 다른 언어/프레임워크로 포팅 👑 최대 시장
**수익성 💰💰💰💰💰 / 난이도 상**

| 타겟 | 시장 | 경쟁 | 기술 |
|---|---|---|---|
| **NestJS/Express** | 🔥🔥🔥 | 거의 없음 | ts-morph |
| **Django/FastAPI** | 🔥🔥🔥 | 거의 없음 | Python `ast` |
| Spring Boot | 🔥🔥 | 일부 | JavaParser |
| Rails | 🔥🔥 | 일부 | Ruby parser |
| Symfony | 🔥 | 없음 | Brain 코드 상당 부분 재사용 |

전략: **아키텍처 설계를 그대로 차용하고 파서만 교체.** React 프론트엔드는 노드/엣지 JSON 포맷만 맞추면 거의 100% 재사용 가능.

⭐ 최우선: **`nest-brain`** — NestJS는 데코레이터 기반 DI라 라라벨보다 분석이 쉽고, npm 생태계라 확산이 빠르며, 시장이 비어있다.

### 💎 5. AI 에이전트 컨텍스트 서버 (MCP 특화)
**수익성 💰💰💰💰 / 난이도 중**

"AI 코딩 에이전트를 위한 코드베이스 인덱서"로 리포지셔닝.

- 차별점: 대부분의 RAG는 **텍스트 임베딩 기반**이지만 우리는 **구조 그래프 기반**.
  "이 함수를 누가 부르는가"는 임베딩이 정확히 못 맞추지만 AST 그래프는 정확하다.
- 수익: 토큰 사용량 과금 또는 시트당 구독 ($20/개발자/월)

### 💎 6. 교육 콘텐츠 & 정보 상품
**수익성 💰💰 / 난이도 하**

| 상품 | 가격대 |
|---|---|
| YouTube "레거시 라라벨 뜯어보기" 시리즈 | 광고 + 협찬 |
| 온라인 강의 "AST로 코드 분석 도구 만들기" | 5~15만원 |
| eBook "라라벨 아키텍처 감사 플레이북" | 3~5만원 |
| 기업 워크숍 (1일) | 200~500만원 |
| 유료 뉴스레터 | $5/월 |

시너지: 콘텐츠로 신뢰 축적 → **2번(감사 서비스) 고객이 인바운드로 유입.**

### 💎 7. 기업 온프레미스 라이선스
**수익성 💰💰💰💰💰 / 난이도 중~상**

타겟: 금융권 · 공공기관 · 대기업 (코드 외부 반출 불가 조직)
구성: 온프레미스 도커 이미지 + 사내 GitLab/Jenkins 연동 + 커스텀 룰 + 연 2회 온사이트 교육 + SLA
가격: 연 2,000만원 ~ 1억원
→ SaaS 1,000명보다 엔터프라이즈 3곳이 더 클 수 있음. 단 영업 사이클 6~12개월.

### 📊 추천 로드맵

```
[0~3개월]  🥇 감사 서비스        투자 0원 · 즉시 수익 · 시장 검증   → 첫 고객 1곳 / 300만원
[3~6개월]  🥈 콘텐츠로 신뢰 축적  감사 사례를 블로그·유튜브로        → 인바운드 월 2건
[6~12개월] 🥉 nest-brain 포팅     현금흐름 유지하며 개발 · 오픈소스   → 스타 500개
[12개월~]  🏆 SaaS 전환           고객 + 평판 기반                  → MRR $3,000
```

> 💡 **핵심 전략: 도구를 파는 게 아니라, 도구로 만든 인사이트를 팔아라.**
> 개발자는 도구에 돈을 안 써도, 의사결정자는 인사이트에 돈을 쓴다.

---

## 5. 한 장 요약

| 질문 | 답 |
|---|---|
| 이게 뭐야? | Laravel 코드베이스를 AST로 정적 분석해 인터랙티브 그래프로 시각화하는 Composer 개발 패키지 |
| 플러그인/스킬/MCP? | **Composer(Laravel) 패키지** + MCP 서버를 옵션으로 내장 |
| API 토큰 필요? | **불필요** — 전 기능 로컬 실행, 코드 외부 전송 없음 |
| 왜 유명해? | 공백 시장 + zero-config + 예쁜 UI + AI 시대 타이밍 + 높은 완성도 + 정직한 문서 |
| 로컬 에이전트에 도움? | **매우** — MCP 툴 10종, 토큰 최적화 컨텍스트, AI 룰 8종 자동 생성 |
| React/PHP로 가능? | **이미 React 19 + PHP 8** 구성. 포크해서 확장하는 편이 효율적 |
| 수익화? | 감사 서비스(즉시) → 콘텐츠 → 타 프레임워크 포팅 → SaaS 전환 |

---

*🤖 이 문서는 카리나(Claude Code)와의 전수조사 대화를 정리한 것입니다. 💖*
