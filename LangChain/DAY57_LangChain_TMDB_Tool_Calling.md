# [DAY_57] LangChain을 활용한 외부 API 연동 및 Custom Tool 설계

### 1. 학습 요약
* **LangChain Tool Calling**: LLM이 외부 API나 함수를 직접 실행할 수 있도록 연결하는 인터페이스 학습
* **TMDB API 연동**: 영화 검색 및 상세 정보 조회를 위한 REST API 호출 로직 구현
* **Agentic Workflow**: 모델이 스스로 도구를 선택하고 실행 결과를 반영하여 답변을 생성하는 과정 이해

### 2. 배운 개념 정리
* **Tool & Toolkit**: LLM이 특정 작업(검색, 계산 등)을 수행할 수 있게 정의된 함수 집합. LangChain에서는 `@tool` 데코레이터를 사용하여 일반 Python 함수를 도구로 변환함.
* **Pydantic 기반 스키마**: LLM이 어떤 인자(Argument)를 전달해야 하는지 명확히 인지할 수 있도록 입력 데이터의 형식을 정의함.
* **Tools Map (Bind Tools)**: 모델에게 사용 가능한 도구 목록을 전달하는 과정. 모델은 사용자의 질문을 분석하여 어떤 도구를 호출할지(Tool Call) 결정함.
* **TMDB(The Movie Database)**: 영화 관련 방대한 데이터를 제공하는 API로, API Key 인증 및 엔드포인트 구조를 파악하는 것이 필수임.

### 3. 코드리뷰 (Custom Tool 설계 예시)
```python
import os
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool

# 1. 커스텀 도구 정의: 날씨 정보 제공 예시 (TMDB 실습 구조와 동일)
@tool
def get_current_weather(location: str):
    """특정 지역의 현재 날씨 정보를 조회합니다."""
    # 실제로는 API 호출 로직이 들어가는 부분
    return f"{location}의 날씨는 맑음, 기온은 22도입니다."

# 2. 모델 생성 및 도구 바인딩 (Tools Map 핵심 부분)
llm = ChatOpenAI(model="gpt-4o-mini")

# 모델이 어떤 도구를 사용할 수 있는지 'bind_tools'를 통해 전달
# 이 과정이 수행되어야 모델의 응답에 'tool_calls' 필드가 포함됨
llm_with_tools = llm.bind_tools([get_current_weather])

# 3. 실행 프로세스
query = "서울 날씨 어때?"
result = llm_with_tools.invoke(query)

# 모델은 답변 대신 '도구를 써야겠다'는 판단(Tool Call)을 내림
print(result.tool_calls) 
```

### 4. 헷갈렸던 점 (Q&A)
* **Q: Tool을 바인딩할 때 왜 dict 형태나 특정 맵핑 구조가 복잡하게 느껴질까요?**
* **A**: 모델은 단순히 함수 이름만 보는 것이 아니라, 함수의 **Docstring(설명문)**과 **Type Hint(인자 타입)**를 보고 사용 여부를 결정하기 때문입니다. 특히 여러 도구를 리스트 형태로 묶어 `bind_tools`에 전달할 때, 각 도구가 고유한 스키마를 가지고 있어야 모델이 혼동하지 않습니다.

### 5. 실무 관점
* **신뢰성(Reliability)**: LLM은 가끔 존재하지 않는 API 파라미터를 생성(Hallucination)할 수 있습니다. 따라서 Tool 내부에서 입력값을 검증하는 로직이 필수적입니다.
* **보안(Security)**: API Key는 절대 코드에 하드코딩하지 않고 `.env` 파일 등 환경변수로 관리해야 합니다. 
* **확장성**: 실무에서는 수십 개의 Tool을 관리해야 하므로, 관련 있는 Tool끼리 그룹화하여 `Toolkit` 단위로 관리하는 패턴을 주로 사용합니다.