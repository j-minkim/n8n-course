# n8n 노드 매핑 레퍼런스 (PRD 6레이어 → 노드)

> PRD 5섹션을 n8n 노드로 옮길 때 참조. **노드명·파라미터 키·typeVersion은 n8n 버전마다 다를 수 있으니** 최종은 에디터에서 확인(또는 n8n-MCP로 정확 조회). 아래는 대표값.

## 레이어별 대표 노드

### ① 트리거 (언제 시작)
| 상황 | 노드 | 핵심 파라미터 |
|---|---|---|
| 정해진 시각/주기 | **Schedule Trigger** (`scheduleTrigger`) | rule.interval(days/weeks·triggerAtHour) |
| 외부 호출/폼 | **Webhook** (`webhook`) / **Form Trigger** | path / httpMethod |
| 메일 수신 | **Email Trigger (IMAP)** (`emailReadImap`) | mailbox·credential |
| 수동(테스트) | **Manual Trigger** (`manualTrigger`) | — |

### ② 입력 (무엇을 받나)
| 소스 | 노드 | 비고 |
|---|---|---|
| 구글시트 | **Google Sheets** (`googleSheets`, op=read) | OAuth credential |
| RSS/뉴스 | **RSS Read** (`rssFeedRead`) | url (자격증명 불필요) |
| 웹/API | **HTTP Request** (`httpRequest`) | url·method·headers |
| 메일 | **Gmail**(read) / IMAP | credential |

### ③ 처리 (변환·판단)
| 작업 | 노드 | 비고 |
|---|---|---|
| 필드 설정/정리 | **Set / Edit Fields** (`set`) | assignments |
| 코드 변환 | **Code** (`code`, JS/Python) | `$input.all()` |
| 조건 분기 | **IF** (`if`) / **Filter** (`filter`) | 조건식 |
| 집계 | **Summarize** (`summarize`) | group/sum (출력키 주의) |
| 합치기 | **Merge** (`merge`) | mode |
| ★AI 판단 | **Anthropic Chat Model** + **Basic LLM Chain**/**AI Agent** | ★별도 Anthropic API 키(종량과금)·Pro 불가. 서브노드 연결(`ai_languageModel`) |

### ④ 출력 (어디로)
| 대상 | 노드 | 비고 |
|---|---|---|
| 메일(OAuth) | **Gmail**(send) (`gmail`) | OAuth |
| 메일(SMTP·회사계정) | **Send Email** (`emailSend`) | SMTP host/port/user/pass(앱비번) |
| 슬랙 | **Slack** (`slack`) | credential |
| 시트 기록 | **Google Sheets**(append/update) | OAuth |

### ⑤ 예외 (실패 처리)
| 작업 | 노드 |
|---|---|
| 에러 시 별도 흐름 | **Error Trigger** (`errorTrigger`) |
| 의도적 중단 | **Stop And Error** (`stopAndError`) |
| 조건 가드 | **IF** / **Wait** |
| 중복·반복 알림 방지 | 이미 알린 키(상품ID 등)를 **Google Sheets**나 워크플로 **Static Data**에 기록 → 다음 실행 때 **IF**로 제외 (모니터링·재고 알림에서 "매 주기마다 같은 알림" 막는 핵심 패턴) |

## 표현식 (환각 1순위 주의)
- 이전 노드 데이터: `{{ $json.필드 }}` — ★**`$json.` 접두사 빠뜨리지 말 것**.
- 한글/공백 필드: `{{ $json["담당자메일"] }}` 대괄호 표기.
- 노드명 참조: `{{ $node["HTTP Request"].json }}` — ★**대소문자 정확히**.
- 날짜: `{{ $now.format('yyyy-MM-dd') }}`(버전별 상이 — 확인).

## 키 3종 (헷갈림 방지)
- **Anthropic API 키**: 워크플로 안 AI 노드 구동(종량과금). console.anthropic.com.
- **n8n API 키**: n8n을 바깥에서 제어(MCP 직접 생성/실행). n8n Settings → n8n API.
- **Claude 구독**: Claude Code 자체. (위 둘과 다름)

## 빌드 원칙
- **노드별 순차 빌드 + 즉시 실행 검증.** 거대 JSON 한 번에 = cascade 에러.
- self-host 발송 한도: Gmail 무료 ~500/일·Workspace ~2,000/일.

---

## 정확 노드 값 · Best Practices
> 출처: **n8n 공식 `.agent/workflows` 가이드**(n8n 소스) + **`czlonkowski/n8n-skills`**(MIT). 깊은 빌드(정확한 파라미터·검증·에러)는 그 자료/MCP에 위임하고, 아래는 설계 단계에서 자주 쓰는 핵심만.

### 노드 정확 typeVersion·파라미터
| 노드 | type | typeVersion | 핵심 |
|---|---|---|---|
| Code | `n8n-nodes-base.code` | 2 | mode·language·jsCode. ★`return [{json:…}]` 필수(`return []`=다음 노드 스킵) |
| Webhook | `n8n-nodes-base.webhook` | 1 | path·httpMethod·responseMode. ★**수신 데이터는 `$json.body` 아래** |
| HTTP Request | `n8n-nodes-base.httpRequest` | 4 | url·method·contentType |
| Schedule Trigger | `n8n-nodes-base.scheduleTrigger` | 1.x | rule.interval |
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | 2.2 | + `ai_languageModel` 서브 연결 |

### AI 노드 구조 (LangChain)
- **AI Agent**(`@n8n/n8n-nodes-langchain.agent`) ← `ai_languageModel`(LLM·필수) + `ai_memory`(선택) + `ai_tool`(선택·여러개).
- LLM: **Anthropic** `n8n-nodes-langchain.lmChatAnthropic`(★별도 Anthropic API 키) / OpenAI `lmChatOpenAi` / Gemini `lmChatGoogleGemini`.
- 연결 타입: `main`(데이터) / `ai_languageModel` / `ai_memory` / `ai_tool`.

### 워크플로 JSON 정본 구조
`{ name, nodes:[{id, name, type, typeVersion, position, parameters, credentials}], connections:{"소스노드":{"main":[[{node,type,index}]]}}, settings:{executionOrder:"v1"} }`
- n8n REST API: `POST /api/v1/workflows` (`-H "X-N8N-API-KEY: <키>"`). 워크플로는 API로 생성 가능. 자격증명은 **강의 실습에선 n8n UI 설정 권장**(API 생성은 버전·플랜·scope에 따라 가능하나 실습은 UI가 안전·단순).

### Best Practices (n8n 공식)
1. **트리거는 워크플로 시작에만** (중간 연결 불가).
2. **Code는 항상 `return`** — `return []`은 워크플로 종료.
3. **AI Agent 도구 ≤ 10~15개** (많으면 신뢰성↓).
4. **복잡 로직은 구조화 워크플로** (AI Agent보다 안정적).
5. **자격증명은 강의 실습에선 n8n UI에서 설정 권장** (API로도 생성 가능하나 버전·플랜·scope 의존 — 실습은 UI가 안전).

### ★환각 주의 (czlonkowski/n8n-skills gotcha)
- Webhook 데이터 = **`$json.body`** 아래.
- **Code Tool**(AI Agent 도구) 반환 = **문자열**(일반 Code 노드의 배열과 다름).
- `$helpers`는 Code 샌드박스에서 **사용 불가**.
- 표현식 노드명 **대소문자**·`$json.` **접두사** 정확히.
