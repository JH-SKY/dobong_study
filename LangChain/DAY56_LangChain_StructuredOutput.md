# [DAY_56] LangChain: Structured Output, Enum 활용 및 대화 요약 파이프라인 구축

### 💡 학습 요약
* **LangChain Structured Output**: LLM의 응답을 사전에 정의된 구조(Schema)에 맞춰 출력하도록 강제하여 데이터 파싱 오류 방지.
* **Enum 활용**: 출력값의 범위를 특정 카테고리로 제한하여 데이터의 일관성 확보.
* **Tool 개념 입문**: LLM이 외부 API나 함수를 호출하여 능동적으로 작업을 수행하게 하는 인터페이스 확인.
* **실습**: 상담 챗봇 구현 및 대화 기록(History) 관리, 종료 시 전체 대화 요약 로직 연동.

---

### 📝 배운 개념 정리

#### 1. Structured Output (구조화된 출력)
* **개념**: LLM은 기본적으로 자유로운 텍스트를 반환하지만, 프로그래밍적으로 데이터를 처리하려면 JSON 같은 구조가 필요함.
* **특징**: `with_structured_output` 메서드를 사용하여 Pydantic 모델이나 JSON Schema 형식으로 답변을 고정함.
* **효과**: 후속 로직(DB 저장, 요약 등)에서 별도의 복잡한 문자열 처리 없이 바로 데이터를 객체처럼 활용 가능.

#### 2. Enum (열거형) 활용
* **개념**: 변수에 들어갈 수 있는 값을 특정 집합으로 제한하는 자료형.
* **활용**: 상담 챗봇에서 고객의 감정 상태를 `[긍정, 보통, 부정]` 중 하나로만 분류하고 싶을 때 사용함. LLM이 엉뚱한 단어를 내뱉지 못하게 가이드라인을 제공함.

#### 3. LangChain Tool (도구) 맛보기
* **개념**: 모델이 스스로 판단하여 특정 기능(계산기, 검색, 데이터베이스 조회 등)을 실행하도록 연결하는 장치.
* **내일의 핵심**: 오늘 배운 구조화된 출력이 '결과'를 만드는 것이라면, Tool은 '실행'을 위한 준비 단계임.

---

### 💻 코드리뷰 (상담 챗봇 및 요약 파이프라인)

```python
from typing import List
from pydantic import BaseModel, Field
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_community.chat_message_histories import ChatMessageHistory

# 1. 출력 구조 정의 (Pydantic)
class ChatSummary(BaseModel):
    summary: str = Field(description="전체 대화 내용을 한 문장으로 요약")
    customer_sentiment: str = Field(description="고객의 최종 감정 상태 (긍정/중립/부정)")

# 2. 챗봇 및 요약 모델 설정
llm = ChatOpenAI(model="gpt-4o")
structured_llm = llm.with_structured_output(ChatSummary)

# 3. 대화 기록 관리 객체
history = ChatMessageHistory()

def run_consulting_chat():
    # [프롬프팅 영역]
    prompt = ChatPromptTemplate.from_messages([
        ("system", "당신은 친절한 도봉캠퍼스 상담원입니다. 사용자의 질문에 답변하세요."),
        MessagesPlaceholder(variable_name="messages"),
    ])
    
    chain = prompt | llm

    # [실행 및 히스토리 저장]
    # 실제 루프 내에서 history.add_user_message(), history.add_ai_message()가 수행됨
    # (예시에서는 단순 흐름만 서술)
    
    # [요약 로직 연동]
    # 대화가 종료(exit)되면 저장된 history를 기반으로 구조화된 요약 수행
    summary_prompt = ChatPromptTemplate.from_messages([
        ("system", "다음 대화 내용을 요약하고 감정을 분석하세요."),
        MessagesPlaceholder(variable_name="messages"),
    ])
    
    summary_chain = summary_prompt | structured_llm
    result = summary_chain.invoke({"messages": history.messages})
    
    print(f"요약: {result.summary}")
    print(f"감정: {result.customer_sentiment}")
```
* **리뷰**: `ChatMessageHistory`를 통해 대화 흐름을 유지하고, 종료 시점에 이 리스트를 `structured_llm`에 넘겨 일관된 형식의 요약본을 뽑아내는 구조입니다. 각 기능을 독립된 체인(Chain)으로 분리하여 관리하는 것이 유지보수에 유리합니다.

---

### ❓ 헷갈렸던 점 (Q&A)

**Q. 왜 그냥 print()로 요약하면 안 되고 굳이 StructuredOutput을 쓰나요?**
**A.** 단순 확인용이라면 상관없지만, 실제 서비스에서는 요약된 내용을 데이터베이스(DB)에 컬럼별로 저장하거나 웹 대시보드에 뿌려줘야 합니다. 이때 AI가 "요약해 드릴게요~" 같은 불필요한 서술어를 붙이면 에러가 나기 때문에, 순수하게 데이터(JSON)만 받기 위해 사용합니다.

---

### 🚀 실무 관점
1. **데이터 파이프라인**: 현업에서는 고객 상담이 끝나면 AI가 자동으로 상담 로그를 분석해 '불만 고객'인지 여부를 Enum으로 분류하고, 담당자에게 슬랙 알림을 보내는 식으로 업무 자동화를 구축합니다.
2. **토큰 관리**: 대화 기록이 너무 길어지면 요약 시 토큰 비용이 많이 발생합니다. 실무에서는 일정 주기마다 중간 요약을 하거나 중요한 맥락만 남기는 `Memory` 관리 기법을 병행합니다.