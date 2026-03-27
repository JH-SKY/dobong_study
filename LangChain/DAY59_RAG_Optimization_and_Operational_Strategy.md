# [DAY_59] RAG 고도화: 검색 최적화, 대화형 아키텍처 및 실무 운영 전략

### 1. 학습 요약
* **LLM의 한계 극복**: Hallucination을 억제하기 위한 Grounding 기술과 사용자 의도 파악을 위한 Clarification 설계 습득
* **검색 고도화(Retrieval Optimization)**: Re-ranking, Hybrid Search, Multi-Query 등 검색 정밀도를 높이는 2단계 전략 이해
* **데이터 관리 전략**: 효율적인 컨텍스트 전달을 위한 Chunking 전략(Parent-Document, Sentence Window) 및 벡터 DB의 CRUD/ID 기반 관리 숙달
* **시스템 아키텍처**: 대화 맥락 유지를 위한 Query Re-writing과 사용자 질문 의도에 따른 Router 기반 분기 처리 설계

---

### 2. 배운 개념 정리

#### 🔹 LLM 설계 원칙 및 한계 대응
* **Hallucination(환각)**: 모델이 확률적으로 다음 단어를 예측하는 특성과 사용자 질문에 반드시 답하려는 UX 강박으로 인해 발생하는 '거짓 답변' 현상
* **Grounding(근거 설정)**: 답변의 범위를 특정 지식(DB/문서)으로 제한하여, 외부 지식이 아닌 주어진 데이터 안에서만 답변하도록 강제하는 기법
* **Clarification(명확화)**: 모호한 질문에 대해 추측 답변을 지양하고, 부족한 정보를 사용자에게 되물어 정확도를 높이는 인터페이스 설계

#### 🔹 검색 및 데이터 구조 최적화
* **Re-ranking(재순위화)**: 1차 검색기로 가져온 다수의 문서 후보군을 '채점 비서(LLM)'가 다시 평가하여 가장 관련성 높은 순서로 재배열하는 정밀화 과정
* **Hybrid Search**: 문맥의 의미를 찾는 **Vector(Dense)** 검색과 특정 키워드를 찾는 **BM25(Sparse)** 검색을 결합하여 검색 누락 최소화
* **Parent-Document Retrieval**: 실제 검색은 작은 조각(Child) 단위로 수행하되, 답변 생성 시에는 부모(Parent) 문서를 참조하여 끊기지 않는 문맥 제공
* **Sentence Window Retrieval**: 검색된 핵심 문장의 앞뒤 문장을 함께 로드하여 LLM에게 풍부한 배경 정보 제공

#### 🔹 대화형 RAG 및 운영 전략
* **Query Re-writing**: 대화 맥락 속의 대명사(예: "그거", "이전 내용")를 포함한 질문을 검색 가능한 **Standalone Question(독립적 질문)**으로 변환
* **ID 기반 벡터 DB 관리**: 문서 삽입 시 반환되는 고유 ID를 활용하여 수정된 문서만 선택적으로 갱신(Incremental Update)하거나 삭제하는 정밀 제어 방식
* **Router(교통경찰)**: 질문의 성격에 따라 RAG 체인, SQL 쿼리 체인, 또는 일반 대화 체인으로 경로를 분기하여 최적의 전문가 모듈이 응답하도록 설계

---

### 3. 코드리뷰 (실무 예시: Re-ranking 및 Query Re-writing 구조)

```python
# 1. Query Re-writing: 맥락이 포함된 질문을 독립적 질문으로 변환
rewrite_prompt = "이전 대화와 질문을 참고하여 검색 엔진에 입력할 최적의 독립적인 질문으로 재작성하세요."
standalone_question = llm.invoke(f"{chat_history} \n 질문: {user_input}")

# 2. 1차 검색 (Similarity Search)
initial_docs = retriever.get_relevant_documents(standalone_question)

# 3. Re-ranking: LLM을 채점 비서로 활용하여 문서 순위 재조정
# 실무 포인트: 모든 문서를 읽기 전 '채점' 단계를 거쳐 품질을 확보합니다.
reranked_docs = llm_reranker.compress_documents(initial_docs, standalone_question)

# 4. 최종 답변 생성
response = qa_chain.invoke({"input_documents": reranked_docs, "question": standalone_question})
```

---

### 4. 헷갈렸던 점 (Q&A)

**Q1. 리랭킹(Re-ranking) 과정에서 왜 비용이 비싼 LLM을 사용하나요?**
* **A**: 리랭킹의 LLM은 '답변'을 쓰는 게 아니라 문서의 '관련성 점수'를 매기는 **채점 비서** 역할을 합니다. 1차 검색(Vector)은 단순 유사도만 보기 때문에, 실제 정답이 포함되었는지 판단하는 똑똑한 눈(LLM)이 필요하기 때문입니다.

**Q2. 대화 기록(History)만 넘겨주면 되는데 왜 굳이 질문 재작성(Rewrite)을 하나요?**
* **A**: 벡터 DB는 "그거 성능 어때?"라는 질문에서 "그거"가 무엇인지 모릅니다. DB에서 데이터를 제대로 뽑아오려면 "그거"를 "구글 AI"처럼 구체적인 단어로 바꾼 **검색용 독립 질문**이 반드시 필요합니다.

**Q3. 벡터 DB 삭제 시 왜 파일명이 아닌 ID(주민번호)를 써야 하나요?**
* **A**: 파일명이나 메타데이터는 '별명'과 같아서 중복될 수 있고 모호합니다. 시스템적으로 특정 위치의 데이터만 정확히 수정하거나 삭제하기 위해서는 `add_documents` 시 생성된 **절대 주소(ID)**를 마스터키로 관리하는 것이 운영 안정성 면에서 필수적입니다.

---

### 5. 실무 관점
* **효율성**: 모든 데이터를 매번 다시 벡터화하는 것은 비용 낭비입니다. **ID 기반의 Incremental Update** 전략을 통해 변경된 부분만 동기화하는 로직을 반드시 구축해야 합니다.
* **유연성**: 사용자 질문은 예측 불가능합니다. **Router 아키텍처**를 도입하여 단순 인사에는 가벼운 모델을, 복잡한 문서 분석에는 RAG 체인을 연결하는 방식이 운영 비용 절감의 핵심입니다.
```