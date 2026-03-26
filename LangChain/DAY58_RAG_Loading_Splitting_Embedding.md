# [DAY_58] RAG 파이프라인 기초: 문서 로드, 분할 및 임베딩 최적화

### 1. 학습 요약
* **데이터 로드(Document Loading)**: 다양한 형식의 데이터(PDF, Text 등)를 모델이 처리할 수 있는 형태로 불러오는 과정
* **문서 분할(Text Splitting)**: 긴 텍스트를 모델의 컨텍스트 제한에 맞춰 의미 있는 단위(Chunk)로 쪼개는 기술
* **임베딩(Embedding)**: 텍스트 데이터를 컴퓨터가 이해할 수 있는 수치(벡터)로 변환하여 의미적 유사성을 계산 가능하게 함

---

### 2. 배운 개념 정리
* **Document Loaders**: 
    * 비정형 데이터를 정형화된 객체로 변환하는 도구입니다. 
    * 단순 텍스트뿐만 아니라 메타데이터(페이지 번호, 출처 등)를 함께 보존하는 것이 핵심입니다.
* **Recursive Character Text Splitter**: 
    * 단순히 글자 수로 자르는 것이 아니라, 문단(`\n\n`), 문장(`\n`), 단어(` `) 순서로 구분자를 우선순위에 두어 텍스트의 맥락을 최대한 보존하며 분할합니다.
* **Vector Embeddings**: 
    * '사과'와 '배'는 텍스트로 보면 다르지만, 벡터 공간에서는 유사한 위치에 놓이게 됩니다. 
    * 딥러닝 모델을 통해 단어의 '의미'를 좌표값으로 변환하는 과정입니다.

---

### 3. 코드리뷰 (LangChain 기준 예시)

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings

# 1. 문서 로드: PDF 파일의 각 페이지를 Document 객체로 변환
loader = PyPDFLoader("learning_material.pdf")
docs = loader.load()

# 2. 문서 분할: chunk_size는 자르는 크기, chunk_overlap은 맥락 연결을 위한 중첩 구간
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,        # 한 덩어리에 최대 500자
    chunk_overlap=50,      # 앞뒤 덩어리와 50자씩 겹치게 하여 문맥 단절 방지
    length_function=len
)
splits = text_splitter.split_documents(docs)

# 3. 임베딩: 텍스트 데이터를 1536차원의(OpenAI 기준) 벡터로 변환
embeddings_model = OpenAIEmbeddings()
sample_vector = embeddings_model.embed_query(splits[0].page_content)

print(f"분할된 문서 수: {len(splits)}")
print(f"임베딩 벡터 차원: {len(sample_vector)}")
```

---

### 4. 헷갈렸던 점 (Q&A)
* **Q: Chunk Overlap은 왜 필요한가요?**
    * **A**: 문장을 자르다 보면 중요한 정보가 앞뒤 덩어리로 나뉘어 의미가 소실될 수 있습니다. 일부를 겹치게(Overlap) 설정하면 질문과 관련된 문맥을 검색 단계에서 더 정확하게 찾아낼 수 있습니다.
* **Q: 왜 한 번에 큰 문서를 다 집어넣으면 안 되나요?**
    * **A**: LLM(거대언어모델)은 한 번에 처리할 수 있는 토큰 제한이 있으며, 정보가 너무 많으면 모델이 핵심 내용을 놓치는 'Lost in the middle' 현상이 발생하기 때문입니다.

---

### 5. 실무 관점
* **성능의 80%는 전처리**: 실제 현업에서 RAG 시스템의 성능은 LLM 자체보다 "얼마나 데이터를 잘 로드하고(Cleaning), 적절한 크기로 잘랐는가"에서 결정됩니다.
* **비용 최적화**: 임베딩은 API 호출 시 비용이 발생하므로, 한 번 변환한 벡터는 'Vector Store(Chroma, Pinecone 등)'에 저장하여 재사용하는 것이 필수적입니다.