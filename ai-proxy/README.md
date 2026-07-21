# 🤖 AI 해석 기능 설치 가이드

봇이 키워드로 못 알아듣는 질문을 진짜 AI(Claude)가 이해하고 답하게 만드는 설정입니다.
**두 가지 계정 준비가 필요해요. 총 15분 정도 걸립니다.**

구조: `앱(GitHub Pages) → Cloudflare Worker(무료 중간 서버) → Claude API`
API 키는 Worker 안에만 저장되고 브라우저에는 절대 노출되지 않아요.

---

## 1단계. Anthropic API 키 발급 (5분)

1. https://console.anthropic.com 접속 → 계정 생성(구글 로그인 가능)
2. 좌측 **Billing** → 결제 카드 등록 후 **크레딧 충전** ($5 정도면 충분히 오래 씁니다)
3. ⚠️ **Billing → Limits에서 월 사용 한도(예: $5)를 꼭 설정**하세요. 예상 밖 과금을 막아줍니다.
4. 좌측 **API Keys** → **Create Key** → 이름 아무거나(예: seondo-bot) → 생성된 `sk-ant-...` 키를 복사해 두세요. (한 번만 보여주니 메모!)

## 2단계. Cloudflare Worker 만들기 (10분)

1. https://dash.cloudflare.com 접속 → 무료 계정 생성
2. 좌측 **Workers & Pages** → **Create** → **Create Worker**
3. 이름을 `seondo-ai` 등으로 정하고 **Deploy** (일단 기본 코드로 배포)
4. **Edit code** 버튼 → 편집기가 열리면 기존 코드를 전부 지우고, 이 폴더의 **worker.js 내용 전체를 붙여넣기** → 우측 상단 **Deploy**
5. Worker 화면으로 돌아가서 **Settings → Variables and Secrets** → **Add**:
   - Type: **Secret** / Name: `ANTHROPIC_API_KEY` / Value: 1단계에서 복사한 `sk-ant-...` 키 → 저장
6. Worker 주소 복사: `https://seondo-ai.<계정이름>.workers.dev` 형태

## 3단계. 앱에 연결

복사한 Worker 주소를 알려주시면 index.html의 `AI_ENDPOINT`에 넣고 배포해 드립니다.
직접 하려면: index.html에서 `const AI_ENDPOINT = ''` 를 찾아 따옴표 안에 주소를 넣고 커밋·푸시하면 끝.

---

## 비용 안내

- Cloudflare Worker: **무료** (하루 10만 요청까지)
- Claude API: 사용한 만큼만 과금. 기본 모델(claude-haiku-4-5) 기준 **질문 1개당 약 5~15원**.
  선생님들이 한 달에 200번 물어봐도 2~3천원 수준입니다.
- 더 똑똑한 답변이 필요하면 Worker의 **Settings → Variables**에 `MODEL` = `claude-opus-4-8` 추가
  (최고 성능 모델, 비용 약 5배 — 질문당 30~80원)

## 동작 방식

- 봇이 키워드로 답할 수 있는 질문 → 지금처럼 **즉시 무료로** 답변 (AI 호출 없음)
- 키워드로 못 알아듣는 질문 → 자동으로 AI에게 전달
- 모든 답변 아래 **"🤖 AI에게 다시 묻기"** 버튼 → 키워드 답변이 엉뚱할 때 AI로 재질문

## 보안 참고

- Worker는 `shussamsujin.github.io`에서 온 요청만 허용(CORS)하고, 질문 길이를 600자로 제한합니다.
- 그래도 주소를 아는 사람은 호출할 수 있으니, **Anthropic 월 한도 설정(1단계 3번)이 최종 안전장치**입니다.
