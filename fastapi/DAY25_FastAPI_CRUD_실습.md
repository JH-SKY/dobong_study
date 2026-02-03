## 1. 학습 요약
UV 가상환경에서 FastAPI를 구축하고, Pydantic 모델을 통한 데이터 검증과 리스트 기반의 상품 CRUD(생성, 조회, 수정, 삭제) 기능을 모듈화하여 구현함.

## 2. 배운 개념 정리
* **UV 환경 및 개발 서버**: `uv run fastapi dev` 명령어를 통해 코드가 수정될 때마다 서버가 자동으로 재시작되는 환경에서 실습을 진행함.
* **Pydantic 모델링**: `BaseModel`을 상속받아 상품의 이름(str)과 가격(int) 타입을 정의함. 서버 입구에서 데이터 형식을 강제하여 에러를 사전에 방지함.
* **APIRouter 모듈화**: `main.py`에 모든 코드를 넣지 않고 `product_api.py`로 기능을 분리하여 관리함. `prefix`와 `tags` 설정을 통해 API 구조를 체계화함.
* **인자 전달 방식**: 객체 생성 시 위치 인자(Positional) 대신 키워드 인자(Keyword)를 사용하여 데이터가 뒤바뀌는 현상을 방지함.

## 3. 코드리뷰
데이터 설계와 API 비즈니스 로직을 분리하여 설계함.

```python
# product.py: 데이터 설계도
from pydantic import BaseModel

class ProductModel(BaseModel):
    # 사용자가 입력할 데이터의 명세
    name: str
    price: int

class Product:
    # 내부 저장용 객체 모델
    def __init__(self, id, name, price):
        self.id = id
        self.name = name
        self.price = price

# product_api.py: CRUD 기능 구현
from fastapi import APIRouter
from .product import ProductModel, Product

router = APIRouter(prefix="/products", tags=["Product"])
products = []
product_id = 0

@router.post("")
def create_product(product_data: ProductModel):
    global product_id
    product_id += 1
    
    # 설계 의도: 데이터 정합성을 위해 키워드 인자로 명확히 매핑함
    new_item = Product(id=product_id, name=product_data.name, price=product_data.price)
    
    products.append(new_item)
    return new_item
```

## 4. 실수/에러와 해결 과정
* **JSON Extra data**: Postman Body에 여러 객체를 나열하여 발생. JSON은 기본적으로 하나의 루트 객체만 허용하므로 개별 전송으로 해결함.
* **데이터 위치 뒤바뀜**: 생성자의 인자 순서(`price`, `name`)를 착각하여 데이터가 꼬임. `name=product_data.name`과 같이 명시적 호출 방식으로 교정함.
* **JSON Syntax Error**: 항목 간 쉼표(`,`) 누락으로 인한 디코드 에러 발생. 에러 메시지의 `loc` 정보를 확인하여 오타를 수정함.

## 5. 실무 관점
실제 서비스에서는 데이터의 무결성이 가장 중요함. Pydantic을 활용한 1차 검증과 파이썬의 명시적인 키워드 인자 사용은 개발자의 실수를 원천 차단하는 실무적인 접근법임. 현재는 메모리에 저장하지만, 추후 이 로직을 유지한 채 저장소만 DB로 교체하면 완성도 높은 API 서비스가 됨.