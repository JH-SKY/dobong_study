# [DAY_64] Multi-Agent System: Plan-and-Execute 패턴의 이해와 구현

### 💡 학습 요약
* **멀티 에이전트(Multi-Agent)**: 하나의 거대 모델이 모든 일을 처리하는 대신, 역할을 분담한 여러 에이전트가 협력하는 구조
* **Plan 패턴 (Planning)**: 복잡한 작업을 바로 실행하지 않고, 해결을 위한 단계별 '계획'을 먼저 수립한 뒤 실행(Execution)하는 아키텍처
* **핵심 가치**: 복잡한 문제의 추론 정확도 향상 및 할루시네이션(Hallucination) 감소

### 📚 배운 개념 정리
* **Planner (계획자)**: 사용자의 복잡한 요구사항을 분석하여 이를 작은 단위의 태스크(Sub-tasks)로 쪼개고 순서를 정하는 역할
* **Executor (실행자)**: Planner가 만든 계획의 각 단계를 실제로 수행하며 도구(Tool)를 사용하거나 데이터를 처리하는 역할
* **Re-planner (재계획자)**: 실행 결과가 예상과 다르거나 실패했을 때, 현재 상황을 반영하여 남은 계획을 수정하는 동적 계획 수립 단계
* **State Management**: 에이전트 간에 계획표와 실행 결과를 공유하기 위한 공유 메모리(State) 관리의 중요성

### 💻 코드리뷰 (실무 예시: LangGraph 스타일의 Plan 패턴)
> 전달받은 코드가 없어, 실무에서 자주 사용되는 LangGraph의 Plan-and-Execute 구조를 예시로 구현했습니다.

```python
from typing import List, TypedDict
from langchain_core.pydantic_v1 import BaseModel, Field

# 1. 상태 정의 (State): 에이전트들이 공유할 데이터 구조
class PlanExecuteState(TypedDict):
    input: str        # 사용자 질문
    plan: List[str]   # 수립된 계획 리스트
    past_steps: List[str] # 실행 완료된 결과 기록
    response: str     # 최종 답변

# 2. 계획 수립 모델 (Planner)
class Plan(BaseModel):
    """실행할 단계별 계획 리스트"""
    steps: List[str] = Field(description="순차적으로 실행할 작업 단계")

# [리뷰] 설계 의도: 
# Planner는 LLM이 구조화된 출력(Structured Output)을 내보내도록 강제하여, 
# 프로그램이 읽을 수 있는 리스트 형태의 계획을 생성합니다.

def plan_step(state: PlanExecuteState):
    # LLM이 state['input']을 보고 Plan 객체를 생성하는 로직 (가상)
    print("LOG: [Planner] 계획 수립 중...")
    return {"plan": ["데이터 검색", "내용 요약", "최종 보고서 작성"]}

def execute_step(state: PlanExecuteState):
    # 현재 계획 중 첫 번째를 가져와 실행
    current_task = state['plan'][0]
    print(f"LOG: [Executor] '{current_task}' 실행 중...")
    # 실행 후 완료된 단계는 past_steps에 저장
    return {"past_steps": [f"완료: {current_task}"], "plan": state['plan'][1:]}

# [리뷰] 작동 원리:
# 1. 사용자의 질문을 받으면 Planner가 전체 로드맵을 작성합니다.
# 2. Executor는 계획이 소진될 때까지 하나씩 미션을 클리어합니다.
# 3. 모든 계획이 끝나면 최종 결과를 반환하는 루프 구조입니다.
```

### ❓ 헷갈렸던 점 (Q&A)
* **Q: 그냥 한 번에 물어보는 것과 Plan 패턴의 차이는 무엇인가요?**
    * **A**: 일반적인 요청(Zero-shot)은 LLM이 생각나는 대로 바로 답을 뱉지만, Plan 패턴은 "먼저 생각하고(Thinking) 행동"하게 만듭니다. 이는 수학 문제 풀이나 논문 요약처럼 과정이 복잡한 작업에서 정답률을 비약적으로 높여줍니다.
* **Q: 계획이 틀리면 어떻게 하나요?**
    * **A**: 그래서 'Re-planning'이 중요합니다. 한 단계를 실행한 결과를 보고 "이 계획으로는 안 되겠는데?"라고 판단하면 즉시 계획을 수정하는 로직을 추가하는 것이 실무적 접근입니다.

### 🚀 실무 관점
* **토큰 효율성**: 모든 지식을 한 번에 다루는 것보다, 단계별로 필요한 데이터만 조회하여 처리하므로 컨텍스트 윈도우 관리에 유리합니다.
* **확장성**: 특정 단계(예: 데이터베이스 조회)만 성능이 떨어진다면, 전체 모델을 바꾸는 게 아니라 해당 단계를 담당하는 'Executor'의 프롬프트나 도구만 개선하면 됩니다. (모듈화 프로그래밍)
* **주의사항**: 루프(Loop)가 무한히 반복되지 않도록 최대 실행 횟수(Recursion Limit)를 반드시 설정해야 비용 폭탄을 막을 수 있습니다.