# [DAY_60] RAG 파이프라인 성능 검증: RAGAS 프레임워크 활용 및 평가 지표 이해

### 1. 학습 요약
* **핵심 주제**: RAG(Retrieval-Augmented Generation) 시스템의 성능을 데이터 기반으로 정량화하는 **RAGAS** 프레임워크 학습
* **주요 내용**: 정답(Ground Truth) 없이도 LLM을 이용해 RAG의 각 단계(검색 및 생성)를 평가하는 'Faithfulness', 'Answer Relevance', 'Context Precision/Recall' 지표 습득

---

### 2. 배운 개념 정리
* **RAGAS (RAG Assessment)**: RAG 파이프라인의 구성 요소인 '검색(Retrieval)'과 '생성(Generation)'을 독립적으로 평가하기 위한 오픈소스 프레임워크
* **RAG Triad (평가 3요소)**:
    * **Faithfulness (충실도)**: 생성된 답변이 검색된 문서 내용에만 기반하는가? (할루시네이션 방지)
    * **Answer Relevance (답변 관련성)**: 생성된 답변이 사용자의 질문에 직접적으로 대답하고 있는가?
    * **Context Relevance (문맥 관련성)**: 검색된 문서들이 질문에 답하는 데 정말 필요한 정보인가?
* **LLM-as-a-Judge**: 사람이 일일이 채점하는 대신, GPT-4와 같은 고성능 LLM을 평가자로 활용하여 대량의 데이터를 빠르게 검증하는 방식

---

### 3. 코드리뷰 (RAGAS 평가 실행 예시)
```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevance, context_precision
from datasets import Dataset

# 1. 평가용 데이터셋 구성 (질문, 답변, 검색문서, 정답 데이터 필요)
data_samples = {
    'question': ['도봉캠퍼스 AI 과정의 특징은?'],
    'answer': ['실무 프로젝트 중심의 AI 개발자 양성 과정입니다.'],
    'contexts' : [['도봉캠퍼스는 실무 프로젝트를 통해 AI 개발자를 양성하는 커리큘럼을 보유하고 있다.']],
    'ground_truth': ['청년취업사관학교 도봉캠퍼스는 실무 중심의 AI 교육을 제공한다.']
}

dataset = Dataset.from_dict(data_samples)

# 2. 지정된 지표로 평가 수행
# faithfulness: 답변이 문서 내 내용인지 확인
# answer_relevance: 질문과 답변의 일치도 확인
score = evaluate(
    dataset,
    metrics=[faithfulness, answer_relevance, context_precision]
)

# 3. 결과 출력 및 분석
print(score.to_pandas())
```
* **설계 의도**: 사용자의 질문에 대해 시스템이 내놓은 답변과 검색해온 문서(Context)를 RAGAS 지표에 입력하여 0~1 사이의 점수로 변환합니다. 이를 통해 어떤 구간에서 성능이 떨어지는지 수치로 파악할 수 있습니다.

---

### 4. 헷갈렸던 점 (Q&A)
* **Q: 정답(Ground Truth) 데이터가 반드시 있어야 하나요?**
    * **A**: RAGAS의 장점은 'Faithfulness'나 'Answer Relevance'처럼 정답 없이(Reference-free) 평가 가능한 지표가 많다는 점입니다. 하지만 'Context Recall'처럼 검색의 정확도를 완벽히 보려면 정답 데이터가 있는 것이 유리합니다.
* **Q: 평가하는 LLM이 틀릴 수도 있지 않나요?**
    * **A**: 맞습니다. 이를 '평가자 편향'이라고 합니다. 따라서 평가용 LLM은 실제 서비스용보다 한 단계 높은 모델(예: GPT-4o)을 사용하는 것이 권장됩니다.

---

### 5. 실무 관점
* **성능 모니터링**: 현업에서는 모델을 한 번 배포하고 끝내는 것이 아니라, RAGAS 점수를 주기적으로 체크하여 프롬프트 수정이나 데이터 증설의 근거로 삼습니다.
* **비용 관리**: 평가 시에도 LLM API를 사용하므로 비용이 발생합니다. 모든 데이터를 평가하기보다 핵심 샘플(Golden Dataset)을 구축하여 반복 테스트하는 전략이 중요합니다.