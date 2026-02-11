### 1. 학습 요약
대량의 데이터를 효율적으로 보여주는 페이지네이션 기법과, 사용자를 식별하고 유지하기 위한 인증(Auth) 및 인가(Authorization) 메커니즘(쿠키, 토큰, 세션)을 학습함.

---

### 2. 배운 개념 정리
* **페이지네이션 (Pagination)**: 수만 개의 데이터를 한 번에 가져오면 서버가 터질 수 있으므로, `offset`과 `limit`을 이용해 서점의 '페이지'처럼 나누어 가져오는 기술임.
* **쿠키 vs 세션 vs 토큰**:
    * **쿠키**: 브라우저(손님) 주머니에 넣어두는 작은 이름표.
    * **세션**: 서버(가게) 장부에 손님 정보를 적어두고 관리하는 방식.
    * **토큰 (JWT)**: 출입증 자체에 정보를 암호화해서 담아두는 방식. 서버가 장부를 일일이 뒤지지 않아도 되어 효율적임.
* **회원가입 & 로그인 (Auth)**:
    * **bcrypt**: 비밀번호를 그대로 저장하지 않고 '해싱'하여 암호화하는 도구.
    * **PyJWT**: 인증된 사용자에게 디지털 출입증(JWT)을 발급하고 검증하는 도구.

---

### 3. 코드리뷰 (예시: 로그인 및 페이지네이션 로직)
```python
# [auth_service.py] bcrypt를 이용한 비밀번호 검증 및 JWT 발급
import bcrypt
import jwt

def login(user_in_db, password_input):
    # 1. 비밀번호 일치 확인 (bcrypt 사용)
    if not bcrypt.checkpw(password_input.encode('utf-8'), user_in_db.password.encode('utf-8')):
        raise Exception("비밀번호가 틀렸습니다.")
    
    # 2. 로그인 성공 시 JWT 토큰 발급 (PyJWT 사용)
    token = jwt.encode({"user_id": user_in_db.id}, "secret_key", algorithm="HS256")
    return token

# [post_repository.py] 페이지네이션 적용 조회
def get_posts(db: Session, skip: int = 0, limit: int = 10):
    # offset(skip)과 limit을 사용하여 필요한 만큼만 잘라오기
    stmt = select(Post).offset(skip).limit(limit)
    return db.scalars(stmt).all()
```

---

### 4. 실수/에러와 해결 과정
* **실수**: 비밀번호를 DB에 평문(Raw Text)으로 저장하려 함.
* **해결**: 보안 사고 방지를 위해 반드시 `bcrypt`로 암호화(Hashing)한 뒤 저장해야 함을 배움.
* **에러**: 페이지네이션 적용 시 `offset`을 설정하지 않아 모든 페이지에서 같은 데이터가 나옴.
* **해결**: `(page - 1) * limit` 공식을 사용하여 현재 페이지에 맞는 데이터를 가져오도록 수정함.

---

### 5. 실무 관점
* **성능**: 페이지네이션은 단순히 화면을 나누는 게 아니라 DB 부하를 줄이는 '필수 최적화' 작업임.
* **보안**: `PyJWT`와 `bcrypt`를 사용하는 이유는 서버가 털리더라도 사용자 비밀번호를 보호하고, 서버의 확장성(Stateless)을 확보하기 위함임.

---

### 6. 성장 포인트 (Retrospective)
* 브라우저(클라이언트)와 서버의 역할 분담을 쿠키와 세션 개념을 통해 명확히 이해함.
* 실무에서 가장 많이 쓰이는 PyJWT와 bcrypt의 조합을 익혀 안전한 로그인 기능을 구현할 수 있는 자신감을 얻음.