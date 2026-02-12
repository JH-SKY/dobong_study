### 1. 학습 요약
오늘 FastAPI를 활용해 **회원(User2) - 상품(Product) - 찜하기(Wishlist2)** 기능을 연결하고, JWT 인증 기반의 '내 찜 목록 조회'까지 백엔드의 핵심 흐름을 완벽히 구축했습니다.

### 2. 배운 개념 정리
- **모델 리팩토링의 연쇄 작용**: `User`에서 `User2`로 이름을 바꾸면 DB 테이블뿐만 아니라 이를 참조하는 모든 외래키(ForeignKey)와 관계(Relationship) 이름도 함께 수정해야 합니다.
- **422 Unprocessable Entity**: Postman에서 보내는 키 이름(예: `password`)과 Pydantic 스키마가 기대하는 이름이 다를 때 발생하는 형식 에러입니다.
- **500 Internal Server Error**: 코드 내부에 존재하지 않는 변수명을 호출하거나(예: `user.password` vs `user.password_hash`), 모델 간의 관계가 꼬였을 때 발생하는 서버 내부 오류입니다.
- **Bearer Token 인증**: 로그인을 통해 얻은 JWT 토큰을 Postman의 Authorization 탭에 넣어줘야만 '나의 데이터'를 안전하게 요청할 수 있습니다.

### 3. 코드리뷰
```python
# product_db/repositories/product_repository.py
# 찜한 상품 목록을 가져오기 위한 조인(Join) 쿼리 구현

def find_wishlist_by_user_id(self, db: Session, user_id: int):
    """
    [설계 의도]
    - Product 테이블과 Wishlist2 테이블을 조인하여 특정 유저가 찜한 '상품 정보'를 가져옴.
    - select(Product)를 통해 결과값으로 상품 객체들을 바로 반환받아 사용 편의성 증대.
    - 대소문자 구분(Wishlist2)과 모델에 정의된 관계 이름(wishlists2)을 정확히 일치시킴.
    """
    stmt = (
        select(Product)
        .join(Product.wishlists2) # Product 모델에 정의된 relationship 이름
        .where(Wishlist2.user_id == user_id) # Wishlist2 모델의 user_id 필드 필터링
    )
    return db.scalars(stmt).all()
```

### 4. 실수/에러와 해결 과정
- **에러**: `TypeError: 'password' is an invalid keyword argument for User2`
- **원인**: `User2` 모델에는 `password_hash`만 있는데, 서비스 로직에서 `User2(password=...)`라고 잘못 전달함.
- **해결**: 모델의 필드명에 맞춰 `password_hash=hashed_password`로 수정하여 DB 저장에 성공함.
- **에러**: 찜 목록 조회 시 401 Unauthorized 발생.
- **해결**: Postman의 Auth 설정을 `Bearer Token`으로 맞추고, 발급받은 최신 토큰을 입력하여 인증 통과.

### 5. 실무 관점
실무에서는 데이터 모델이 수시로 변합니다. 이때 단순히 에러를 고치는 것에 급급하기보다, **입력(Schema) -> 처리(Service) -> 저장(Model)**으로 이어지는 데이터의 이름표(Naming)가 일관되게 유지되고 있는지 확인하는 것이 유지보수의 핵심입니다.

---
