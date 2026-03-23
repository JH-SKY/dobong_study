# [DAY_55] LangChain 기초: LLM 연결 및 Chain 구조의 이해

### 1. 학습 요약

- **LangChain 프레임워크 입문**: LLM(거대언어모델)을 효율적으로 활용하기 위한 라이브러리 학습
- **기본 구성 요소**: PromptTemplate, LLM, OutputParser의 역할 이해
- **LCEL(LangChain Expression Language)**: `|` (파이프) 연산자를 이용한 체인 구성 및 데이터 흐름 실습

### 2. 배운 개념 정리

- **PromptTemplate (프롬프트 템플릿)**
  - 사용자의 입력을 변수로 받아 사전에 정의된 프롬프트 형식에 주입하는 틀
  - 단순 텍스트 입력보다 구조화된 요청을 LLM에 전달 가능
- **LLM (OpenAI 연결)**
  - `ChatOpenAI` 클래스를 통해 GPT-4o 등 외부 API 모델과 연결
  - Temperature 등 파라미터를 조절하여 응답의 창의성 제어
- **OutputParser (출력 파서)**
  - LLM의 응답(객체 형태)에서 순수 텍스트(String)만 추출하거나, JSON/List 형태로 가공하는 역할
- **Chain (체인)**
  - 여러 구성 요소를 하나로 묶어 실행하는 단위
  - `프롬프트 | 모델 | 파서` 형태의 파이프라인 구조를 가짐

### 3. 코드리뷰 (LCEL 기초 실습)

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 1. 모델 설정 (OpenAI API 키 설정 필요)
model = ChatOpenAI(model="gpt-4o", temperature=0.7)

# 2. 프롬프트 템플릿 생성
# {topic} 부분에 사용자 입력이 동적으로 주입됨
prompt = ChatPromptTemplate.from_template("{topic}에 대해 아주 짧고 간결하게 설명해줘.")

# 3. 출력 파서 생성 (응답 데이터에서 텍스트만 추출)
output_parser = StrOutputParser()

# 4. 체인 구성 (LCEL 문법)
# 입력 -> 프롬프트 구성 -> 모델 실행 -> 결과 텍스트 추출의 파이프라인
chain = prompt | model | output_parser

# 5. 실행 (invoke 함수 사용)
response = chain.invoke({"topic": "랭체인의 장점"})
print(response)
```

### 4. 헷갈렸던 점 (Q&A)

- **Q. 입력값이 어떻게 전달되고 결과가 텍스트로 나오나요?**
  - **A. 데이터 흐름도**:
    1. **Input**: `{"topic": "..."}` 형태의 딕셔너리가 들어감
    2. **Prompt**: 템플릿 내의 `{topic}` 변수가 입력값으로 치환됨
    3. **LLM**: 완성된 문장을 AI에게 전달하고 `AIMessage`라는 객체 응답을 받음
    4. **Parser**: 복잡한 `AIMessage` 객체 안에서 우리가 읽을 수 있는 `.content` (문자열)만 쏙 뽑아냄
- **Q. 왜 굳이 파이프(`|`)를 쓰나요?**
  - **A.** 코드가 직관적입니다. 마치 공장의 컨베이어 벨트처럼 데이터가 왼쪽에서 오른쪽으로 흐르는 것을 한눈에 볼 수 있어 유지보수가 쉽습니다.

### 5. 실무 관점

- **모듈화**: 실무에서는 프롬프트만 수정하거나 모델을(GPT -> Claude) 교체해야 하는 상황이 빈번함. LangChain은 각 단계를 분리해 두었으므로 특정 부분만 교체하기 매우 유리함.
- **비용 관리**: 무분별한 프롬프트 호출은 비용 발생의 원인. 체인 구조 내에서 입력을 사전에 검증하거나(Validator), 결과를 정제하는 파서를 두어 토큰 낭비를 방지함.
