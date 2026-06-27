# CS 분류·요약 프롬프트 (Lab4-AI)

이 프롬프트는 `lab4-complete-AI포함.json`의 **CS 분류요약 체인**(`@n8n/n8n-nodes-langchain.chainLlm`) 노드 `prompt(text)` 파라미터에 그대로 들어가요. 아래 본문을 복사해서 사용하면 돼요.

## 프롬프트 본문 (체인 노드 prompt에 동일하게 입력)

```
너는 홈쇼핑 CS 분류 도우미야. 다음 고객 문의를 [배송 / 환불 / 상품문의 / 기타] 중 정확히 하나로 분류하고, 한 줄로 요약해줘.
반드시 아래 JSON 형식으로만 답해:
{"분류":"<배송|환불|상품문의|기타>","요약":"<한 줄 요약>"}
문의: {{ $json.문의내용 }}
```

## 사용법

- 입력 데이터: `CS문의.csv`(구글시트 `CS문의` 탭)의 각 행. `{{ $json.문의내용 }}`로 문의 본문을 주입해요.
- 모델 노드: **Anthropic Chat Model**(`@n8n/n8n-nodes-langchain.lmChatAnthropic`)을 `ai_languageModel`로 체인에 연결해요. (main 연결 아님)
- 모델명(예: claude-3-5-sonnet 계열)은 단정하지 말고 **강의 전 본인 로컬 `npx n8n`에서 dry-run 1회로 선택·확인**해요. (UI/모델 목록은 버전마다 다를 수 있어요.)
- 출력: 체인 결과(JSON 문자열)를 `분류결과` 탭에 append 해요. JSON으로 강제했기 때문에 후속 파싱이 쉬워요.

## Anthropic API 키 (사실 고정값)

- n8n 워크플로 안의 AI 노드는 **별도 Anthropic API 키**(console.anthropic.com · 종량과금)가 필수예요.
- **Claude Pro 구독($20/월, USD)으로는 구동되지 않아요.** Claude Pro는 claude.ai 챗 + 설계·디버깅 보조 전용이에요.
- 실습에서는 **회사가 제공한 API 키**를 n8n Credentials(Anthropic API)에 입력하고, 수강생이 직접 핸즈온해요.

## 흔한 실수 & 해결

- **라벨이 제각각으로 나옴**: 허용 라벨을 고정하지 않으면 모델이 자기 마음대로 라벨을 붙여요. 프롬프트에 `[배송 / 환불 / 상품문의 / 기타] 중 하나로만` 분류하라고 명시하세요. JSON 형식 강제로 후속 파싱도 쉬워져요.
- **401 / authentication 에러**: Claude Pro 구독으로 착각했거나 API 키를 안 넣은 경우예요. **회사가 제공한 Anthropic API 키**를 Credentials에 입력하세요.
