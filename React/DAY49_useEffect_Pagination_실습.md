# [DAY_49] React useEffect를 활용한 API 데이터 페칭 및 페이지네이션 구현

### 📝 학습 요약
* **useEffect Hook**: 컴포넌트의 생명주기에 따라 외부 API를 호출하고 사이드 이펙트를 관리하는 방법 학습
* **Axios 비동기 통신**: `async/await` 구문을 사용하여 외부 데이터(DummyJSON)를 안정적으로 가져오는 프로세스 이해
* **Pagination 로직**: `limit`와 `skip` 파라미터를 조절하여 대량의 데이터를 구간별로 나누어 렌더링하는 설계 방식 습득
* **조건부 렌더링 및 방어 코드**: 삼항 연산자를 활용한 페이지 경계 처리(첫/마지막 페이지) 구현

---

### 📚 배운 개념 정리
* **useEffect의 의존성 배열 (Dependency Array)**: 
  * 배열 내에 특정 값(예: `skip`)을 넣으면, 해당 값이 변경될 때마다 `useEffect` 내부의 함수가 재실행됨
  * 이를 통해 사용자 인터랙션(버튼 클릭)에 따른 실시간 데이터 업데이트가 가능함
* **API Query Parameter (`limit`, `skip`)**:
  * `limit`: 한 페이지에 보여줄 데이터의 개수
  * `skip`: 건너뛸 데이터의 개수. (현재 페이지 - 1) * limit의 공식을 가짐
* **비동기 함수(Async/Await) 처리**:
  * JavaScript는 싱글 스레드이므로, 데이터를 기다리는 동안 화면이 멈추지 않도록 비동기 처리가 필수적임

---

### 🔍 코드리뷰
```javascript
import axios from "axios";
import React, { useState, useEffect } from "react";

const PostsJh = () => {
  // 1. 상태 관리: 서버 데이터를 저장할 posts와 페이징 기준점인 skip 선언
  const [posts, setPosts] = useState([]);
  const [skip, setSkip] = useState(0);

  useEffect(() => {
    // 2. [설계 의도] 데이터 페칭 함수 정의
    // 비동기 통신을 위해 async를 사용하며, try-catch 문을 추가하면 에러 핸들링에 더 효과적임
    const fetchPosts = async () => {
      const response = await axios.get(
        `https://dummyjson.com/posts?limit=10&skip=${skip}`,
      );
      const data = response.data;
      setPosts(data.posts); // 3. 응답받은 데이터를 상태값에 업데이트하여 리렌더링 유도
    };

    fetchPosts();
    // 4. [알람 설정] skip 값이 변경될 때마다 이 useEffect가 다시 실행됨 (반응형 데이터 로드)
  }, [skip]);

  return (
    <div className="card">
      <div className="flex">
        {/* 처음으로/마지막으로: skip 값을 상수로 고정하여 빠른 이동 구현 */}
        <button className="button" onClick={() => setSkip(0)}>처음으로</button>

        {/* 5. [방어 로직] 현재 skip 상태를 확인하여 비정상적인 데이터 요청(음수 등) 차단 */}
        <button
          className="button"
          onClick={
            skip === 0
              ? () => alert("첫페이지 입니다.")
              : () => setSkip(skip - 10)
          }
        >
          이전
        </button>

        <button
          className="button"
          onClick={
            skip === 250
              ? () => alert("마지막 페이지 입니다.")
              : () => setSkip(skip + 10)
          }
        >
          다음
        </button>

        <button className="button" onClick={() => setSkip(250)}>마지막으로</button>
      </div>

      {/* 6. [데이터 렌더링] 고유한 key값(post.id)을 부여하여 React의 최적화 규칙 준수 */}
      {posts.map((post) => (
        <li key={post.id} className="card">
          <div>게시글번호 : {post.id}</div>
          <div>제목 : {post.title}</div>
          <div>작성자 : {post.userId}</div>
          <div>조회수 : {post.views}</div>
        </li>
      ))}
    </div>
  );
};

export default PostsJh;
```

---

### ❓ 헷갈렸던 점 (Q&A)

**Q1: 페이지 이동 버튼마다 각각 다른 API 호출 함수를 만들어야 하나요?**
* **처음 생각**: '이전', '다음', '처음으로', '마지막으로' 버튼마다 각각 `fetchPosts1`, `fetchPosts2`처럼 별도의 함수를 6개 정도 만들어야 한다고 생각했습니다.
* **깨달은 점**: API 주소의 특정 부분(`skip=${skip}`)을 **변수(State)**로 처리하고, `useEffect`의 의존성 배열에 그 변수를 넣어두면, 함수 하나만으로도 모든 페이지 이동을 자동 처리할 수 있다는 것을 배웠습니다. 코드가 훨씬 간결해지고 유지보수가 쉬워졌습니다.

**Q2: "마지막 페이지입니다"라는 경고창 로직은 어디에 위치해야 하나요?**
* **처음 생각**: API를 가져오는 함수 내부에서 체크해야 하는지, 아니면 화면을 그리는 곳에서 해야 하는지 혼란스러웠습니다.
* **깨달은 점**: 사용자의 잘못된 클릭(범위를 벗어난 이동)을 **'입구'에서부터 차단**하기 위해, 버튼의 `onClick` 이벤트 핸들러 내부에 삼항 연산자(조건문)를 배치하는 것이 가장 직관적이고 효율적임을 알게 되었습니다. 데이터 요청을 보내기 전에 미리 검사하여 불필요한 네트워크 통신을 방지할 수 있습니다.

---

### 🚀 실무 관점
1.  **로딩 상태 관리 (Loading State)**: 데이터가 서버에서 오는 동안 사용자에게 "읽어오는 중..."이라는 표시를 해주는 로딩 스피너 처리가 실제 서비스에서는 필수입니다.
2.  **컴포넌트 분리**: 현재는 한 파일에 버튼과 리스트가 다 있지만, 실무에서는 `PaginationButton`과 `PostList`로 컴포넌트를 분리하여 재사용성을 높입니다.
3.  **에러 핸들링**: 네트워크 단절이나 API 서버 오류 시 `alert`이나 에러 페이지를 보여주는 `try-catch` 패턴 적용이 권장됩니다.