# [DAY_51] React & FastAPI 풀스택 연동: 인증 기반 게시판 시스템 구현

### 1. 학습 요약
* **Full-Stack Architecture**: FastAPI(Backend)와 React(Frontend) 간의 비동기 통신 구현
* **Authentication & Authorization**: JWT 또는 세션을 활용한 로그인/회원가입 및 접근 제어
* **Database CRUD**: SQLAlchemy(또는 Tortoise)를 활용한 게시글 DB 연동 및 작성/삭제 기능 구현
* **Security**: 로그인 여부에 따른 게시글 수정/삭제 권한 검증 로직 적용

### 2. 배운 개념 정리
* **FastAPI (Backend)**: Python 기반의 고성능 웹 프레임워크로, 비동기 처리가 강점이며 Swagger를 통한 API 문서화가 자동 생성됨
* **React State & Effect**: 로그인 상태(Token)를 관리하고, 페이지 로드 시 서버로부터 데이터를 fetch하여 화면을 렌더링함
* **CORS (Cross-Origin Resource Sharing)**: 서로 다른 포트(예: 3000번과 8000번)에서 통신할 때 발생하는 보안 정책을 FastAPI 미들웨어로 해결
* **Token-Based Auth**: 사용자가 로그인하면 서버에서 검증된 토큰을 발급하고, 이후 요청 시 Header에 담아 보내 서버가 사용자를 식별함

### 3. 코드리뷰 (실무형 예시)
* **Backend: 게시글 삭제 권한 검증 (FastAPI)**
```python
@app.delete("/posts/{post_id}")
async def delete_post(post_id: int, current_user: User = Depends(get_current_user), db: Session = Depends(get_db)):
    # 1. DB에서 해당 게시글 조회
    post = db.query(Post).filter(Post.id == post_id).first()
    
    # 2. 게시글 존재 여부 및 작성자 일치 확인 (인가 보안)
    if not post:
        raise HTTPException(status_code=404, detail="게시글을 찾을 수 없습니다.")
    if post.author_id != current_user.id:
        raise HTTPException(status_code=403, detail="삭제 권한이 없습니다.")
    
    # 3. 삭제 처리
    db.delete(post)
    db.commit()
    return {"message": "성공적으로 삭제되었습니다."}
```
* **Frontend: 인증 헤더를 포함한 요청 (React)**
```javascript
const handleDelete = async (postId) => {
  const token = localStorage.getItem("access_token"); // 저장된 토큰 호출
  
  const response = await fetch(`http://localhost:8000/posts/${postId}`, {
    method: 'DELETE',
    headers: {
      'Authorization': `Bearer ${token}`, // Bearer 타입 인증 헤더 전송
    }
  });

  if (response.ok) {
    alert("삭제 완료!");
    // 상태 업데이트 로직 추가...
  }
};
```

### 4. 헷갈렸던 점 (Q&A)
**A:**  **"왜 로그인을 했는데 자꾸 401 Unauthorized 에러가 뜰까요?"**
* **이유**: 대부분 Frontend에서 `fetch`나 `axios` 요청을 보낼 때 `Headers`에 토큰을 담지 않았거나, Backend의 CORS 설정에서 `Allow_Headers`에 인증 관련 헤더가 누락되었을 때 발생합니다. 또한, 토큰의 유효기간이 만료되었는지도 체크해야 합니다.

### 5. 실무 관점
* **보안의 핵심**: 클라이언트(React)단에서의 검증은 '사용자 편의'를 위한 것이며, 실제 '보안'은 서버(FastAPI)에서 이루어져야 함. (서버에서 반드시 유저 ID와 작성자 ID를 대조해야 함)
* **REST API 설계**: 자원(Post)에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE)를 명확히 구분하여 사용하는 것이 협업의 기본임
* **에러 핸들링**: 서버 에러 발생 시 사용자에게 500 에러를 그대로 노출하기보다, 적절한 알림 메시지로 유도하는 처리가 중요함