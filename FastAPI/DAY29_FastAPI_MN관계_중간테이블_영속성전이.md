### 1. 학습 요약
오늘은 FastAPI의 서비스-레포지토리 패턴을 바탕으로 M:N(다대다) 관계를 중간 테이블(PostTag)로 직접 설계하고, 영속성 전이(Cascade)와 지름길 기능(AssociationProxy)을 활용하여 복잡한 데이터를 효율적으로 처리하는 실무적인 설계 방법을 배웠습니다.

### 2. 배운 개념 정리
* **M:N 관계와 중간 테이블**: 고기와 이름표 사이를 연결하는 '라벨지(PostTag)'를 모델즈에 직접 테이블로 정의하는 방식입니다. 단순 연결을 넘어 '언제 붙였는지' 같은 추가 정보를 담기에 유리합니다.
* **영속성 전이 (Cascade)**: 부모(게시글)가 창고에서 나갈 때 매달린 라벨지(PostTag)들도 운명을 같이하게 만드는 설정입니다. 특히 인연이 끊긴 데이터를 자동으로 지워주는 `delete-orphan`이 핵심입니다.
* **AssociationProxy**: 라벨지를 일일이 뒤지지 않고 고기에서 이름표로 바로 직행하는 '지름길 티켓'입니다. `post.tags`라고만 해도 건너편의 태그 목록을 바로 볼 수 있어 코드가 깔끔해집니다.
* **Depends(get_db)**: 창고 열쇠(DB 세션)를 내가 직접 챙기는 게 아니라, 지배인(FastAPI)이 알아서 가져다주고 요리가 끝나면 수거해가는 '자동 관리 서비스'입니다. 리소스 누수를 막아주는 안전장치입니다.

### 3. 코드리뷰
```python
# [설계 의도] 
# 1. 태그가 있으면 가져오고, 없으면 새로 찍어내는 유연한 로직 구현.
# 2. 영속성 전이를 활용해 게시글 하나만 저장해도 연결된 데이터가 줄줄이 사탕처럼 저장되게 설계.

def create_post_with_tags(self, db: Session, data: PostCreateWithTags):
    # 게시글 객체 생성 (아직 영수증만 있는 상태)
    new_post = Post(title=data.title, content=data.content)

    with db.begin():
        for name in data.tags:
            # 창고에 같은 이름표가 있는지 확인
            tag = tag_repository.find_by_name(db, name)

            # 없으면 실시간으로 6번, 7번 이름표를 새로 생성
            if not tag:
                tag = Tag(name=name)
                tag_repository.save(db, tag)
                db.flush() # 다음 번호를 미리 따놓기 위한 작업

            # 라벨지(중간테이블)를 만들어 고기와 이름표를 묶음
            post_tag_link = PostTag(post=new_post, tag=tag)
            new_post.post_tags.append(post_tag_link) # 리스트에 추가하면 나중에 한꺼번에 저장됨

        # 대장(Post)만 저장해도 Cascade 덕분에 라벨지까지 세트로 저장됨
        post_repository.save(db, new_post)

    db.refresh(new_post) # DB가 발급한 진짜 ID와 생성시간을 다시 읽어옴
    return new_post
```

### 4. 실수/에러와 해결 과정
* **리프레시의 오해**: `db.refresh()`를 안 하면 내 메모리 속 데이터에는 ID나 생성시간이 비어있을 수 있습니다. 사용자에게 "글 번호 00번이 생성되었습니다"라고 보여주려면, 저장 직후 창고 문을 다시 열어(`refresh`) 번호를 확인해야 합니다.
* **의존성 주입**: `get_db()`를 직접 호출하면 창고 문을 안 닫는 실수를 할 수 있지만, `Depends`를 쓰면 지배인이 알아서 닫아주므로 코드가 훨씬 안전해집니다.

### 5. 실무 관점
* **효율적인 통신**: 글을 작성(`POST`)하고 나서 다시 조회(`GET`)를 요청하는 게 아니라, 라우터가 응답할 때 완성된 데이터(ID 포함)를 바로 돌려주는 것이 네트워크 자원을 아끼고 사용자 경험을 높이는 실무 표준입니다.
* **확장성**: `PostTag`를 모델로 정의해두면 나중에 "이 태그를 누가 가장 먼저 붙였나?" 같은 데이터가 필요해질 때 테이블 구조를 엎지 않고 바로 컬럼만 추가해서 대응할 수 있습니다.