# [DAY_53] 실시간 AI 상호작용: OpenAI 챗봇 통합 및 데이터 스트리밍 구현

### 1. 학습 요약
* **핵심 내용**: 기존 게시판 프로젝트에 OpenAI API를 연동하여 AI 챗봇 기능을 추가하고, 응답 지연을 해소하기 위한 스트리밍(Streaming) 기술 적용
* **기술 스택**: React(Frontend), Node.js/Express(Backend), OpenAI SDK, Server-Sent Events(SSE) 개념

### 2. 배운 개념 정리
* **OpenAI API 통합**: GPT 모델을 활용하여 사용자의 질문에 답변을 생성하는 기능을 서버(Back-end) 측에 구축함. API Key 보안을 위해 클라이언트가 아닌 서버에서 호출하는 구조가 필수적임.
* **스트리밍(Streaming)의 필요성**: AI가 긴 답변을 생성할 때 한 번에 결과를 받으려면 수 초에서 수십 초의 대기 시간이 발생함. 이를 방지하기 위해 데이터가 생성되는 즉시 한 글자씩 클라이언트에 전달하는 방식임.
* **Server-Sent Events (SSE)**: 서버에서 클라이언트로 실시간 데이터를 푸시(Push)하는 기술. HTTP 연결을 유지하면서 텍스트 조각을 지속적으로 전송함.

### 3. 코드리뷰
```javascript
// Backend: OpenAI 스트리밍 응답 구현 예시
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

app.post('/api/chat', async (req, res) => {
    const { message } = req.body;

    // 스트리밍 응답 설정 (SSE 헤더 설정)
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    const stream = await openai.chat.completions.create({
        model: 'gpt-3.5-turbo',
        messages: [{ role: 'user', content: message }],
        stream: true, // 스트리밍 모드 활성화
    });

    for await (const chunk of stream) {
        const content = chunk.choices[0]?.delta?.content || "";
        if (content) {
            // 조각 데이터를 클라이언트로 즉시 전송
            res.write(`data: ${JSON.stringify({ content })}\n\n`);
        }
    }
    res.end();
});
```
* **설계 의도**: `stream: true` 옵션을 통해 OpenAI로부터 데이터를 비동기 이터레이터 형식으로 받음. `res.write`를 사용하여 전체 작업이 끝나기 전에 응답을 부분적으로 전송함으로써 사용자 체감 속도를 대폭 개선함.

### 4. 헷갈렸던 점 (Q&A)
* **Q: 스트리밍 시 프론트엔드에서 데이터는 어떻게 받나요?**
* **A**: 일반적인 `axios` 보다는 `fetch` API의 `ReadableStream`을 사용하거나, SSE 전용 라이브러리를 활용하여 끊임없이 들어오는 데이터를 상태(State)에 누적시켜 화면에 그려줍니다.

### 5. 실무 관점
* **UX 최적화**: 챗봇 서비스에서 스트리밍 유무는 사용자 이탈률에 직격타를 줍니다. 타이핑 효과(Typing Effect)와 결합하여 AI가 실시간으로 생각하고 답변하는 듯한 느낌을 주는 것이 실무 표준입니다.
* **비용 관리**: OpenAI 모델 사용 시 토큰 단위로 과금되므로, 프롬프트의 길이를 제한하거나 캐싱(Caching) 전략을 세워 불필요한 API 호출을 줄이는 설계가 중요합니다.