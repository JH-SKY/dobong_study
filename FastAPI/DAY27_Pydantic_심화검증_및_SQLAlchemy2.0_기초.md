1. 학습 요약
Pydantic의 field_validator를 통한 비즈니스 로직 검증과 SQLAlchemy 2.0의 현대적인 ORM 방식을 활용하여, 실제 DB와 연동되는 안정적인 상품 관리 시스템의 기반을 구축함.

2. 배운 개념 정리 (중학생도 이해하기 쉽게!)
- 필드 간 검증 (field_validator): "할인가는 원가보다 작아야 한다"처럼 두 값을 비교해야 할 때 쓰는 안전장치예요. 하나만 봐서는 모르는 규칙을 잡아냅니다.
- SQLAlchemy 2.0: 파이썬 코드를 DB 언어로 번역해주는 '최신판 통역사'예요. 예전 방식보다 더 명확하게 "이거 선택해(select)", "이거 저장해(commit)"라고 말하는 게 특징입니다.
- 커밋(Commit): 메모리에서만 만지작거리던 데이터를 DB라는 진짜 창고에 '영구 저장'하고 도장을 쾅 찍는 마지막 단계예요.
- 프레임워크: 요리할 때 모든 도구와 기본 양념이 준비된 '밀키트 주방' 같아요. 우리는 레시피에 맞춰 고기(로직)만 넣으면 됩니다.

3. 코드리뷰 (SQLAlchemy 2.0 모델 및 검증 예시)
# 설계 의도: DB 테이블 구조를 정의하고, 입력 단계에서 논리적 오류를 차단함
from sqlalchemy.orm import Mapped, mapped_column, DeclarativeBase
from pydantic import BaseModel, field_validator, ValidationInfo

# 1. DB 모델 (SQLAlchemy 2.0 방식)
class Base(DeclarativeBase):
    pass

class Product(Base):
    __tablename__ = "products"
    # Mapped와 mapped_column으로 타입을 명확하게 선언 (실무 스타일)
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    price: Mapped[int]
    discount_price: Mapped[int]

# 2. 검증 스키마 (Pydantic)
class ProductCreate(BaseModel):
    price: int
    discount_price: int

    @field_validator("discount_price")
    @classmethod
    def check_discount(cls, v: int, info: ValidationInfo) -> int:
        # info.data를 통해 먼저 들어온 price 값과 비교함
        if "price" in info.data and v >= info.data["price"]:
            raise ValueError("할인 금액은 원가보다 낮아야 합니다.")
        return v # 검증된 값은 반드시 반환!

4. 실수/에러와 해결 과정
- 문제: DB에 데이터를 넣었는데 조회가 안 됨.
- 원인: SQLAlchemy 2.0에서는 session.add()만으로는 부족하고, session.commit()을 호출해야 실제 DB에 반영됨을 간과함.
- 해결: 데이터 변경 작업 후 반드시 session.commit()을 호출하여 트랜잭션을 확정 짓는 습관을 가짐.

5. 실무 관점
- SQLAlchemy 2.0의 Mapped 방식을 쓰면 개발 툴(IDE)이 타입을 정확히 인식해서 오타를 줄여주고 자동 완성을 도와줍니다.
- commit() 직전의 Flush 과정을 이해하면, DB에 가기 전에 에러를 미리 잡는 튼튼한 코드를 짤 수 있습니다.