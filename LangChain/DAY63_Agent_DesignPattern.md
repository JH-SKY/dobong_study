# [DAY_63] LLM 추론 전략: ReAct 에이전트 및 핵심 디자인 패턴의 이해

### 1. 학습 요약
* **LLM 에이전트 설계**: 모델이 스스로 추론(Reason)하고 행동(Act)하는 **ReAct** 패턴의 구조와 원리 학습
* **소프트웨어 디자인 패턴**: 유지보수와 확장성을 고려한 코드 구조 설계 원칙 이해

---

### 2. 배운 개념 정리

#### ❶ ReAct (Reason + Act) 에이전트
* **개념**: LLM이 당면한 문제를 해결하기 위해 '생각(Thought)'을 먼저 하고, 외부 도구를 사용하는 '행동(Action)'을 수행한 뒤, 그 결과물인 '관찰(Observation)'을 다시 추론에 반영하는 프레임워크입니다.
* **작동 프로세스**:
    1.  **Thought**: 질문을 분석하고 다음 단계를 계획합니다.
    2.  **Action**: 검색, 계산기 등 외부 Tool을 호출합니다.
    3.  **Observation**: 실행 결과를 받아옵니다.
    4.  **Repeat**: 문제가 해결될 때까지 위 과정을 반복합니다.

#### ❷ 디자인 패턴 (Design Patterns)
* **싱글톤 패턴 (Singleton)**: 애플리케이션 전체에서 인스턴스를 단 하나만 생성하도록 보장하는 패턴입니다. (예: DB 연결 설정, 로그 기록기)
* **팩토리 패턴 (Factory)**: 객체 생성 로직을 별도 클래스로 분리하여, 클라이언트가 구체적인 클래스 타입을 몰라도 객체를 생성할 수 있게 합니다.
* **전략 패턴 (Strategy)**: 실행 중에 알고리즘(전략)을 선택할 수 있게 하여 코드의 유연성을 높입니다.

---

### 3. 코드리뷰: ReAct 에이전트 및 패턴 응용
> ReAct 로직의 핵심인 LangChain 스타일의 에이전트 구성 예시입니다.

```python
# ReAct 에이전트 구성을 위한 기본 구조 예시
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI

# 1. 도구(Tool) 정의: 에이전트가 'Act'할 수 있는 수단
def get_weather(city: str):
    return f"{city}의 날씨는 맑음(25도)입니다."

tools = [
    Tool(
        name="Weather_Tool",
        func=get_weather,
        description="특정 도시의 현재 날씨 정보를 가져올 때 사용합니다."
    )
]

# 2. 에이전트 초기화: ReAct(Zero-shot) 패턴 적용
# LLM은 이제 'Thought' 단계를 거쳐 'Weather_Tool'을 쓸지 스스로 판단합니다.
agent = initialize_agent(
    tools, 
    OpenAI(temperature=0), 
    agent="zero-shot-react-description", 
    verbose=True
)

# 3. 실행
# 내부적으로 [Thought -> Action -> Observation -> Final Answer] 순으로 작동함
agent.run("도봉구 날씨 어때?")
```

---

### 4. 헷갈렸던 점 (Q&A)
* **Q: ReAct와 일반 프롬프트 엔지니어링의 차이는 무엇인가요?**
    * **A**: 일반 프롬프팅은 한 번에 답을 내놓으라고 요구하지만, ReAct는 **'단계별 추론'**과 **'외부 도구 연동'**을 명시적으로 결합하여 할루시네이션(환각)을 줄이고 최신 데이터 활용을 가능케 합니다.
* **Q: 디자인 패턴은 AI 개발에서 왜 중요한가요?**
    * **A**: AI 서비스는 모델 교체, 프롬프트 변경이 잦습니다. 이때 디자인 패턴이 적용되지 않으면 코드 전체를 뜯어고쳐야 하지만, 패턴을 쓰면 유연한 대처가 가능합니다.

---

### 5. 실무 관점
* **ReAct의 한계**: 무한 루프에 빠질 위험이 있어 실무에서는 `max_iterations`를 설정하는 것이 필수입니다.
* **패턴 활용**: LLM 서비스 개발 시, 다양한 모델(GPT, Claude, Gemini)을 쉽게 갈아 끼우기 위해 **'전략 패턴'**이나 **'어댑터 패턴'**을 적극적으로 도입하여 벤더 종속성을 줄입니다.