# [DAY_62] LangGraph: 에이전트의 기억(Memory)과 체크포인트(Checkpoint) 관리

## 1. 학습 요약
* **핵심 주제**: LangGraph 기반 리액트 에이전트(ReAct Agent)의 메모리 구조 및 영속성 구현
* **주요 내용**: 
    * 에이전트의 단기/장기 메모리 개념 이해
    * `Checkpointer`를 활용한 대화 상태 저장 및 복구
    * Thread ID를 이용한 사용자별 세션 관리

---

## 2. 배운 개념 정리

### ① Short-term Memory (단기 메모리)
* **정의**: 하나의 대화 세션(Thread) 내에서 직전의 대화 맥락을 기억하는 능력입니다.
* **특징**: LangGraph에서는 `State` 객체를 통해 노드 간 데이터를 전달하며, 현재 진행 중인 루프 내의 정보를 유지합니다.

### ② Long-term Memory (장기 메모리)
* **정의**: 세션이 종료된 후에도 사용자의 선호도, 과거 기록 등을 별도의 데이터베이스에 저장하여 다음 접속 시 활용하는 능력입니다.
* **구현**: 외부 DB(PostgreSQL, Redis 등)와 연동하여 에이전트가 "과거에 나눴던 대화"를 인지하게 합니다.

### ③ Checkpoint (체크포인트)
* **정의**: 에이전트의 상태(State)를 특정 시점에 스냅샷으로 저장하는 기능입니다.
* **역할**: 
    * 에러 발생 시 마지막 지점부터 재시작 가능
    * '사람의 승인(Human-in-the-loop)' 단계에서 에이전트의 실행을 일시 중지했다가 다시 재개할 때 필수적입니다.

---

## 3. 코드리뷰

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.prebuilt import create_react_agent

# 1. 체크포인터 초기화 (인메모리 방식)
# 실무에서는 데이터 유실 방지를 위해 SqliteSaver 등을 사용합니다.
memory = MemorySaver()

# 2. 리액트 에이전트 생성
# 도구(tools)와 모델을 결합하고, checkpointer를 등록하여 메모리 기능을 활성화합니다.
agent_executor = create_react_agent(
    model, 
    tools, 
    checkpointer=memory
)

# 3. 실행 설정 (Thread ID 부여)
# thread_id를 기준으로 서로 다른 대화 세션을 구분합니다.
config = {"configurable": {"thread_id": "user_1"}}

# 에이전트 실행
input_message = {"messages": [("user", "내 이름은 도봉이야.")]}
for event in agent_executor.stream(input_message, config):
    print(event)

# 동일한 thread_id로 재질문 시 앞선 대화를 기억함
input_message_2 = {"messages": [("user", "내 이름이 뭐라고?")]}
# 에이전트는 체크포인트를 통해 "도봉"이라는 이름을 기억하고 답변합니다.
```

---

## 4. 헷갈렸던 점 (Q&A)

**Q. Thread ID를 다르게 설정하면 어떻게 되나요?**
**A.** LangGraph는 `thread_id`를 기준으로 체크포인트를 저장합니다. ID가 다르면 에이전트는 이를 아예 다른 사용자로 인식하여 이전 대화 내용을 공유하지 않습니다.

**Q. MemorySaver는 영구적으로 저장되나요?**
**A.** 아니요. `MemorySaver`는 휘발성 메모리에 저장되므로 프로그램이 종료되면 데이터가 사라집니다. 실무 포트폴리오를 만들 때는 `SqliteSaver`나 `PostgresSaver`를 사용하는 것을 권장합니다.

---

## 5. 실무 관점
* **신뢰성 있는 시스템 구축**: AI 에이전트는 도중 흐름이 끊기는 경우가 많습니다. 체크포인트를 사용하면 에러가 나도 처음부터 다시 연산할 필요가 없어 클라우드 비용을 절감하고 사용자 경험을 개선할 수 있습니다.
* **개인화 서비스**: 장기 메모리를 활용해 사용자의 스타일(예: "짧게 대답해줘")을 기억하게 함으로써 고도화된 커스텀 AI 서비스를 구현할 수 있습니다.