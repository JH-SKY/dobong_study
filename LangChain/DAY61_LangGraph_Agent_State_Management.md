# [DAY_61] LangGraph & AI Agent: 워크플로우 설계 및 상태 관리 기초

### 1. 학습 요약
* **AI Agent 프레임워크**: 단순 일회성 호출이 아닌, 루프(Loop)와 조건부 로직을 포함한 복잡한 AI 워크플로우 설계 도구인 LangGraph 학습
* **Graph Structure**: State(데이터 바구니), Node(작업 단위), Edge(흐름 제어)의 유기적 연결 구조 이해
* **신뢰성 확보**: 리듀서(Reducer)를 통한 데이터 누적과 조건부 에지(Conditional Edge)를 활용한 품질 검증 프로세스 실습

---

### 2. 배운 개념 정리
* **State (TypedDict)**: 노드 간에 공유되는 '공용 장부' 역할을 함. 각 노드는 이 장부에서 데이터를 꺼내 쓰고, 수정된 내용을 다시 장부에 기록함
* **Node (함수)**: 특정 임무를 수행하는 독립적인 '일꾼'. 입력으로 State를 받고, 업데이트할 State의 일부를 딕셔너리 형태로 반환함
* **Edge (통로)**: `START` 노드에서 시작하여 각 일꾼 사이를 잇는 길. `END`에 도달하면 프로세스가 종료됨
* **Reducer (Annotated & operator.add)**: 기본적으로 State는 새로운 값으로 덮어씌워지나, 대화 기록처럼 데이터를 쌓아야 할 경우 리듀서를 사용하여 기존 값에 내용을 추가(누적)함
* **Conditional Edge**: '검토자(Reviewer)' 노드의 결과에 따라 다시 수정 단계로 돌아갈지, 최종 답변을 내보낼지 결정하는 '분기점' 역할

---

### 3. 코드리뷰 (실무 예시: 검토 루프가 포함된 에이전트)
```python
from typing import TypedDict, Annotated
import operator
from langgraph.graph import StateGraph, START, END

# 1. State 정의 (장부 규격 설정)
class AgentState(TypedDict):
    # operator.add를 통해 메시지가 덮어쓰이지 않고 리스트에 추가됨
    messages: Annotated[list, operator.add] 
    status: str

# 2. Node 정의 (일꾼 로직)
def researcher_node(state: AgentState):
    print("---연구원: 자료 조사 중---")
    return {"messages": ["조사된 데이터: LangGraph는 유연합니다."], "status": "review_needed"}

def reviewer_node(state: AgentState):
    print("---검토자: 품질 확인 중---")
    # 단순 예시로 상태값에 따라 통과 여부 결정
    if "유연" in state["messages"][-1]:
        return "pass"
    return "fail"

# 3. Graph 빌드 (공장 설계)
workflow = StateGraph(AgentState)

workflow.add_node("researcher", researcher_node)

# 시작점 설정
workflow.add_edge(START, "researcher")

# 조건부 에지 설정: 리뷰 결과에 따라 다음 행선지 결정
workflow.add_conditional_edges(
    "researcher",
    reviewer_node,
    {
        "pass": END,            # 통과 시 종료
        "fail": "researcher"    # 실패 시 다시 연구원에게
    }
)

# 4. Compile (설계도 기반 앱 생성)
app = workflow.compile()
```
* **리뷰 포인트**: 
    * `Annotated[list, operator.add]`를 사용하여 이전 대화 맥락을 유지하도록 설계함
    * `add_conditional_edges`를 통해 무한 루프에 빠지지 않도록 논리적 분기점을 명확히 설정함
    * 노드 함수는 사이드 이펙트 없이 오직 `State`를 받아 수정본을 `return`하는 구조를 유지함

---

### 4. 헷갈렸던 점 (Q&A)
* **Q: 왜 굳이 State를 따로 정의하고 return 문에서 딕셔너리를 쓰나요?**
    * **A**: LangGraph는 '불변성(Immutability)' 원칙을 따릅니다. 원본 데이터를 직접 수정하는 것이 아니라, 노드가 반환한 '차이점'을 그래프가 장부에 반영하는 방식입니다. 이를 통해 에러 발생 시 추적이 용이하고 상태 복구(Checkpointer)가 가능해집니다.
* **Q: KeyError가 발생하는데 원인이 무엇인가요?**
    * **A**: `State` 클래스에 정의한 키 이름(예: `topic`)과 노드 함수 내에서 호출하는 이름(`state['user_text']`)이 다를 때 주로 발생합니다. 항상 장부(State)의 이름표를 먼저 확인해야 합니다.

---

### 5. 실무 관점
* **신뢰성 있는 AI**: 실무에서는 LLM의 답변을 그대로 사용자에게 내보내지 않습니다. 오늘 배운 **리뷰 루프**를 통해 답변의 팩트 체크나 금칙어 필터링 단계를 거치는 것이 에이전트 설계의 기본입니다.
* **비용 관리**: 무한 루프(`fail`이 반복되는 경우)는 API 비용 폭증의 원인이 됩니다. 실무에서는 `max_iterations`를 설정하여 일정 횟수 이상 실패 시 에러를 반환하도록 안전장치를 마련해야 합니다.