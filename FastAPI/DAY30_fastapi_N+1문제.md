# 1. 학습 요약
SQLAlchemy 환경에서 발생하는 성능 저하의 주범인 N+1 문제를 정의하고, 이를 해결하기 위한 다양한 로딩 전략(지연 로딩, 즉시 로딩, 복합 로딩)의 메커니즘과 실무 적용 기준을 학습함.

# 2. 배운 개념 정리
### 🧩 N+1 문제 (The N+1 Query Problem)
* **정의**: 메인 데이터 1개를 가져오는 쿼리(1) 외에, 연관된 데이터를 가져오기 위해 데이터 개수(N)만큼 추가 쿼리가 발생하는 현상이야.
* **원인**: ORM의 기본 설정인 **지연 로딩(Lazy Loading)** 때문이야. "필요할 때 가져오겠다"는 의도는 좋지만, 반복문(for문) 안에서 연관 객체에 접근하면 DB 호출이 폭발해.

### ⚡ 로딩 전략 (Loading Strategies)
1. **지연 로딩 (Lazy Loading)**
   - 필요 시점에 쿼리 실행. 단건 조회에는 효율적이지만 목록 조회 시 N+1 유발.
2. **즉시 로딩 (Eager Loading)**
   - **joinedload()**: SQL의 `JOIN`을 사용하여 한 번의 쿼리로 연관 데이터까지 싹 긁어와. (주로 1:1, N:1 관계)
   - **selectinload()**: 메인 ID들을 모아 `IN` 절로 두 번째 쿼리를 날려. (주로 1:N, N:M 관계, 데이터 중복 방지에 유리)
3. **복합 관계 로딩 (Chained/Multiple Loading)**
   - `joinedload(A).joinedload(B)` 처럼 기차놀이하듯 연속으로 연결하거나, 여러 관계를 동시에 로드할 수 있어.

# 3. 코드리뷰
```python
# [Repository] N+1 문제를 방지한 목록 조회 설계
from sqlalchemy.orm import joinedload

def find_all(self, db: Session, keyword: str = None, category_name: str = None):
    # 1. joinedload를 사용하여 Product와 연관된 Category를 '즉시 로딩' 설정
    stmt = select(Product).options(joinedload(Product.category))
    
    if keyword:
        stmt = stmt.where(Product.name.contains(keyword))
    
    if category_name:
        # 필터링을 위한 join과 데이터 로드를 위한 joinedload를 적절히 배분
        stmt = stmt.join(Product.category).where(Category.name == category_name)
        
    stmt = stmt.order_by(Product.id.desc())
    return db.scalars(stmt).all()
```
* **설계 의도**: 상품 목록을 뿌려줄 때 카테고리 이름이 반드시 필요하므로, `joinedload`를 통해 DB 방문 횟수를 1회로 고정함. `join()`은 필터링(WHERE)을 위해 사용하고, `options()`는 데이터를 실제 객체에 담기 위해 사용함.

# 4. 실수/에러와 해결 과정
* **헷갈렸던 점**: "단순히 `join()`만 걸면 N+1이 해결되겠지?"라고 생각하기 쉬워.
* **해결**: `join()`은 SQL 쿼리상에서 테이블을 붙여 필터링할 때 쓰는 거고, 파이썬 객체에 연관 데이터를 바로 채워 넣으려면 `joinedload()` 같은 **로더(Loader)** 설정을 명시해야 한다는 점을 확실히 함.

# 5. 실무 관점
* **성능 최적화의 1순위**: 실무에서 API 응답 속도가 느려지면 십중팔구 N+1 문제야. 
* **로딩 전략 선택**: 데이터가 적은 1:1 관계는 `joinedload`가 편하지만, 댓글처럼 데이터가 많은 1:N 관계는 `selectinload`를 써야 메모리 낭비를 줄이고 DB 성능을 지킬 수 있어.
