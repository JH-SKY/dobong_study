📚 학습 기록 저장소 (dobong_study)에 맞춰서 정리했습니다.

1. 학습 요약
데이터를 한 번에 다 만들어서 보내는 것이 아니라, 생성되는 즉시 조금씩 끊어서 실시간으로 클라이언트에게 전달하는 SSE와 스트리밍 기술을 학습함.

2. 배운 개념 정리
- SSE (Server-Sent Events): 서버가 클라이언트에게 일방적으로 데이터를 계속 밀어 넣어주는 '단방향 통로'예요. 마치 TV 생중계를 보는 것과 비슷하죠.
- 스트리밍 응답: 100페이지짜리 책을 다 쓸 때까지 기다렸다가 주는 게 아니라, 한 페이지씩 써지는 대로 바로바로 읽어주는 방식이에요.
- Generator와 yield: 파이썬에서 데이터를 한꺼번에 `return`하지 않고, `yield`를 사용해 하나씩 '생산'해내는 특수한 함수예요. 스트리밍의 핵심 엔진이죠.

3. 코드리뷰 (예시 코드 기반)
FastAPI를 활용하여 1초마다 숫자를 보내는 스트리밍 서버 예시입니다.

from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI()

# 데이터를 생성하는 '발전기' 역할의 함수
async def data_generator():
    for i in range(1, 6):
        await asyncio.sleep(1)  # 1초간 작업 중인 척 기다림
        # SSE 형식에 맞춰 "data: 내용\n\n" 구조로 전달
        yield f"data: {i}번째 메시지 도착!\n\n"

@app.get("/stream")
async def stream_data():
    # 생성기(generator)를 StreamingResponse에 담아 반환
    # media_type을 text/event-stream으로 설정하는 것이 SSE의 약속!
    return StreamingResponse(data_generator(), media_type="text/event-stream")

"""
[리뷰]
1. data_generator: yield를 사용해 메모리 효율을 높였습니다. 데이터를 리스트에 담아 한 번에 보내는 게 아니라 그때그때 보냅니다.
2. StreamingResponse: FastAPI가 제공하는 클래스로, 연결을 끊지 않고 계속 데이터를 흘려보낼 수 있게 해줍니다.
3. 실무 팁: 변수명은 content_streamer나 event_publisher처럼 역할이 명확하게 지어주는 것이 좋습니다.
"""

4. 실수/에러와 해결 과정
- 흔한 실수: SSE 응답 형식을 지키지 않는 것.
- 증상: 브라우저나 클라이언트에서 데이터를 받긴 하는데, 이벤트로 인식을 못 함.
- 해결: 반드시 `data: `로 시작해서 `\n\n`(줄바꿈 두 번)으로 끝나야 하나의 메시지로 인식됩니다. 형식을 꼭 맞춰주세요!

5. 실무 관점
- 왜 쓰는가?: ChatGPT처럼 답변이 긴 AI 서비스에서 답변이 다 나올 때까지 빈 화면만 보여주면 사용자는 "고장 났나?"라고 생각합니다. 한 글자씩 출력되는 '타이핑 효과'를 주어 사용자 체감 속도를 높이는 데 필수적입니다.
- 주의점: 연결이 계속 유지되기 때문에 서버 자원을 계속 점유합니다. 동시 접속자가 많을 경우 서버 성능 최적화가 중요합니다.