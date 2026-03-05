# 📚 학습 기록: 실전 웹 기술 (Axios & DOM Control)

### 1. 학습 요약
사용자의 입력(Form)을 받아 서버와 통신(AJAX/Axios)하고, 받아온 데이터를 화면에 동적으로 그리는(DOM) 전체 프로세스를 실습함.

### 2. 배운 개념 정리
* **Axios & Promise**: 서버와 데이터를 주고받을 때 사용하는 도구. `await`를 써서 데이터가 도착할 때까지 기다리는 '약속(Promise)' 시스템을 활용함.
* **AJAX**: 화면 전체를 새로고침 하지 않고 필요한 데이터만 쏙 가져오는 통신 기술.
* **DOM & 노드 (Node)**: HTML의 각 태그를 '노드'라고 부르며, 이를 자바스크립트로 직접 만들거나(`createElement`) 자식으로 붙이는(`appendChild`) 조작법.
* **이벤트 (Event)**: 사용자의 클릭이나 폼 제출 같은 '사건'. 이를 감시하다가 로직을 실행함.
* **Form**: 사용자의 데이터를 묶어서 보내는 그릇. 기본 새로고침 동작을 막고 AJAX로 연결하는 것이 핵심.

### 3. 코드리뷰 (실전 로직)
```javascript
// [설계 의도] 폼 제출 시 새로고침 없이 영화 데이터를 화면에 출력함

// 1. DOM 노드 찾아오기
const movieList = document.querySelector("#movie-list");

const loadMovies = async function () {
  // 2. AJAX 통신 (Axios): 서버 창고에서 영화 데이터를 배달받음
  const response = await axios.get(URL, config);
  
  // 3. 기존 노드 비우기 (중복 방지)
  movieList.innerHTML = "";

  response.data.results.forEach((movie) => {
    // 4. 새로운 DOM 노드 생성
    const card = document.createElement("div");
    card.className = "movie-card";
    
    // 5. 노드 조립 (부모 노드에 자식 노드 붙이기)
    const title = document.createElement("h3");
    title.textContent = movie.title;
    card.appendChild(title);
    movieList.appendChild(card);
  });
};
```

### 4. 헷갈렸던 점 (Q&A)
* **Q: 401 에러가 왜 나나요?**
    * **A**: "당신 누구야?"라는 뜻입니다. API KEY가 비어있거나 `Authorization` 헤더 설정이 잘못되었을 때 발생하니 출입증을 확인해야 합니다.
* **Q: 왜 innerHTML = "" 을 하나요?**
    * **A**: 이전에 그려진 영화 카드(노드)들을 싹 지워야 새로운 검색 결과가 깔끔하게 보이기 때문입니다.

### 5. 실무 관점
* **AI 서비스 설계**: 사용자가 질문을 입력(Form)하면 AI 서버에 요청(AJAX)을 보내고, 답변이 오면 화면에 예쁘게 출력(DOM)하는 것이 현재 배우는 AI 서비스 개발의 기본 골격입니다.