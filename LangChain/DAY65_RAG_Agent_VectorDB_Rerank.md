# [DAY_65] RAG 에이전트 고도화: Vector DB 임베딩 및 Re-rank 기반 노드 워크플로우

### 1. 학습 요약
* **핵심 내용**: 비정형 데이터를 벡터화하여 저장하는 Vector DB 구축과, 사용자 질의에 최적화된 답변을 제공하는 RAG(Retrieval-Augmented Generation) 시스템 설계
* **주요 성과**: 단순 검색을 넘어 5개의 논리적 노드를 통한 조건부 분기(Routing)와 검색 결과의 정밀도를 높이는 Re-rank 모델 적용 실습

### 2. 배운 개념 정리
* **임베딩(Embedding) & 벡터 DB**: 텍스트 데이터를 고차원의 수치 벡터로 변환하는 과정. 변환된 데이터는 Pinecone, ChromaDB 등 벡터 DB에 저장되어 '의미적 유사도' 기반 검색에 활용됨
* **RAG 에이전트**: LLM의 지식 한계를 극복하기 위해 외부 지식 베이스(DB)를 참조하여 답변을 생성하는 지능형 시스템
* **Re-rank(재정렬)**: 1차로 검색된 다수의 문서 중, 질문과의 관련성이 가장 높은 순서대로 점수를 매겨 다시 정렬하는 과정. Hallucination(환각) 방지에 필수적임
* **노드(Node) 기반 워크플로우**: 전체 로직을 독립적인 단계(검색, 평가, 생성, 검증 등)로 분리하여 관리하는 구조적 설계 방식

### 3. 코드리뷰 (LangGraph 기반 RAG 에이전트 예시)
```python
# RAG 에이전트 핵심 로직: 5단계 노드 구성 예시

def retrieve_document(state):
    """[Node 1] 질문과 유사한 문서를 Vector DB에서 검색"""
    question = state["question"]
    documents = retriever.get_relevant_documents(question)
    return {"documents": documents}

def grade_documents(state):
    """[Node 2] 리랭크(Re-rank)를 통한 문서 적합성 검증 및 분기 결정"""
    # 검색된 문서와 질문 사이의 관련성 점수 산출 (Re-ranker 활용)
    filtered_docs = reranker.compress_documents(state["documents"], state["question"])
    
    # 관련 문서가 없으면 '웹 검색' 노드로, 있으면 '답변 생성' 노드로 이동
    if not filtered_docs:
        return "web_search"
    return "generate"

def generate_answer(state):
    """[Node 3] 선별된 문서를 바탕으로 LLM 답변 생성"""
    context = state["documents"]
    response = llm_chain.invoke({"context": context, "question": state["question"]})
    return {"answer": response}

def check_hallucination(state):
    """[Node 4] 생성된 답변이 문서 내용에 근거하는지 검증"""
    # 답변 검증 로직 수행 후 통과 시 종료, 실패 시 재생성 시도
    pass

def web_search(state):
    """[Node 5] 내부 DB에 정보가 없을 경우 외부 검색 엔진 활용"""
    # Search API 호출 로직
    pass
```

### 4. 헷갈렸던 점 (Q&A)
* **Q: 왜 단순히 검색만 하지 않고 리랭크(Re-rank) 단계를 거치나요?**
  * **A**: 벡터 유사도 검색은 '단어의 의미'는 비슷할지 몰라도 질문의 '의도'에 정확히 부합하지 않는 문서를 가져올 때가 많습니다. 리랭크 모델(예: BGE-Reranker)은 연산량은 많지만 훨씬 정밀하게 순위를 재조정하여 AI의 엉뚱한 답변을 사전에 차단합니다.
* **Q: 노드를 나누어 분기하는 이유는 무엇인가요?**
  * **A**: 모든 질문에 대해 똑같은 프로세스를 타는 것보다, 질문의 난이도나 정보 유무에 따라 '검색 후 바로 답변' 혹은 '검색 실패 시 웹 서칭' 등으로 경로를 최적화하여 비용과 속도를 개선하기 위함입니다.

### 5. 실무 관점
* **데이터 신선도 유지**: 실무에서는 임베딩된 데이터가 구식이 되지 않도록 주기적으로 벡터 DB를 업데이트(Upsert)하는 파이프라인 관리가 매우 중요함
* **비용 최적화**: 모든 문서에 리랭크를 적용하면 API 비용과 지연 시간(Latency)이 상승함. 따라서 1차 검색 결과를 적절한 개수(Top-K)로 제한하는 튜닝이 필수적임
* **보안성**: 벡터 DB에 기업의 기밀 데이터를 저장할 경우, 접근 권한 제어와 암호화 처리가 아키텍처 설계 단계에서 고려되어야 함
```