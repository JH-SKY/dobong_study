# 📚 오늘 배운 내용 정리 (2026-02-04)

### 1. 학습 요약
FastAPI를 활용한 레이어드 아키텍처 설계와 Pydantic Field/Annotated를 이용한 데이터 검증 로직 학습.

### 2. 배운 개념 정리
* **Query Parameter & 응답 기능:** URL의 쿼리 스트링을 통해 데이터를 전달받는 방식과 클라이언트 요청에 따른 서버의 응답 제어 흐름.
* **데이터 검증 및 필드 제어 (Validation):** 사용자가 입력한 데이터의 유효성(범위, 필수값 등)을 Pydantic의 Field와 Annotated를 활용해 입구에서 차단하는 방법.
* **관심사 분리 (Separation of Concerns):** 각 모듈이 하나의 역할(라우팅, 로직, DB 접근)만 수행하도록 분리하여 코드 복잡도를 낮추는 원칙.
* **레이어드 아키텍처:** 실무 프로젝트의 유지보수성을 위해 Router, Service, Repository 계층으로 나누어 설계하는 구조.

### 3. 코드리뷰
상품 엔티티에 데이터 검증과 관심사 분리 개념을 적용해본 코드

```python
from typing import Annotated
from pydantic import BaseModel, Field

# [Schema Layer] - 데이터 검증 및 필드 제어
class ProductCreate(BaseModel):
    # 상품명: 1자 이상 50자 이하 제한
    name: Annotated[str, Field(min_length=1, max_length=50, description="상품명")]
    # 원가: 0보다 큰 정수만 허용
    price: Annotated[int, Field(gt=0, description="상품 원가")]
    # 할인 금액: 0 이상의 정수 (원가보다 클 수 없는 로직은 서비스에서 처리)
    discount_price: Annotated[int, Field(ge=0, description="할인 금액")]
    stock: Annotated[int, Field(default=1, ge=0)]
    category: str | None = "전체"

# [Service Layer] - 비즈니스 로직 처리
# 데이터 규격 외에 '할인가가 원가보다 작아야 함' 같은 도메인 규칙 검증
def validate_product_logic(product: ProductCreate):
    if product.discount_price > product.price:
        return "할인 금액이 원가보다 클 수 없음"
    return True
```

### 4. 실수/에러와 해결 과정
* **문제 상황:** 모든 검증 로직과 데이터 처리를 Router 한곳에 작성하여 코드가 비대해짐.
* **해결 방법:** 관심사 분리 원칙에 따라 규격 검증은 Pydantic이, 비즈니스 규칙은 Service 레이어가 담당하도록 분리하여 가독성 확보.
* **사고 과정:** 단순히 에러를 막는 것에 그치지 않고, Annotated를 활용해 코드 자체가 명세서 역할을 하도록 설계함.

### 5. 실무 관점
* **데이터 무결성:** 사용자의 잘못된 입력을 방어하는 Validation은 실제 서비스 운영 시 시스템의 안정성을 결정짓는 핵심 요소.
* **확장성:** 레이어드 아키텍처를 적용하면 향후 요구사항 변경이나 DB 교체 시 수정 범위를 최소화할 수 있어 실무에서 매우 선호하는 구조임.