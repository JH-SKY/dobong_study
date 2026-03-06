# 📚 학습 기록: 리액트 기초 레이아웃 & Props 정복

### 1. 학습 요약
오늘은 리액트의 뼈대인 JSX 문법부터 시작해서, 화면을 레고처럼 조립하는 컴포넌트 구조, 그리고 부모가 자식에게 데이터를 넘겨주는 Props 실습까지 한 바퀴 싹 돌았음.

### 2. 배운 개념 정리
1. **JSX (JavaScript XML)**:
   - JS 안에서 HTML을 쓰는 문법. 1. 무조건 하나의 부모 태그로 감싸야 함, 2. `class` 대신 `className` 쓰기, 3. 스타일은 `backgroundColor`처럼 낙타 등(Camel Case) 모양으로 작성하기가 핵심임.
2. **SPA (Single Page Application)**:
   - 페이지를 새로고침하며 통째로 갈아 끼우는 게 아니라, 필요한 부분(컴포넌트)만 쏙쏙 바꾸는 방식. 사용자 입장에서 엄청 빠르고 부드러움.
3. **Virtual DOM (연습장)**:
   - 브라우저에 바로 그리는 게 아니라 가상의 연습장에 먼저 그려보고, 바뀐 부분만 실제 화면에 반영함. 효율성이 미쳤음.
4. **.map() (변신 기계)**:
   - 배열에 든 데이터를 하나씩 꺼내서 HTML 조각(JSX)으로 변신시켜줌. 이때 `item` 같은 변수는 내가 마음대로 지어주는 이름표임.
5. **Props (데이터 택배)**:
   - 부모가 자식에게 데이터를 전달하는 유일한 수단. 자식은 이걸 받아서 화면에 보여주기만 하면 됨.

### 3. 코드리뷰: 레이아웃 & Props 실습
```jsx
// 1. 자식 컴포넌트: 부모가 준 데이터를 받아서 보여줌
function MenuTitle(props) {
  return (
    // 2. 설계 의도: 제목을 고정하지 않고 부모가 주는 대로 바꾸기 위해 props 사용
    <div style={{ backgroundColor: "#cfe2f3", padding: "10px" }}>
      {props.title}
    </div>
  );
}

// 3. 부모 컴포넌트: 전체를 조립하고 데이터를 관리함
function App() {
  const menuData = "오늘의 추천 메뉴";
  
  return (
    <div className="wrapper">
      {/* 4. 데이터 전달: title이라는 이름의 택배 상자에 menuData를 담아서 보냄 */}
      <MenuTitle title={menuData} />
      
      <nav>네비게이션 바</nav>
      
      {/* 5. Flexbox: middle-container 안의 자식들을 가로로 나란히 배치함 */}
      <div className="middle-container" style={{ display: "flex" }}>
        <main style={{ flex: 3 }}>메인 콘텐츠 구역</main>
        <aside style={{ flex: 1 }}>사이드바</aside>
      </div>
      
      <footer>Footer</footer>
    </div>
  );
}
```

### 4. 헷갈렸던 점 Q&A
- **Q: JSX 안에서 왜 `if`나 `for`를 못 쓰는지?**
  - **A**: JSX `{ }` 안에는 '값'이 바로 나와야 하는데, `if`나 `for`는 동작 명령일 뿐임. 대신 결과값을 반환하는 삼항 연산자나 `.map()`을 쓰는 게 리액트의 규칙임.
- **Q: 별점이나 이모티콘 처리는 어디서 하나?**
  - **A**: 이건 '보여주기용' 로직이라 프론트엔드에서 처리하는 게 맞음. 나중에 디자인 바뀌었을 때 서버 안 건드려도 되니까 훨씬 편함.

### 5. 실무 관점
- **시맨틱 태그 활용**: 단순히 `div`만 쓰지 말고 `header`, `nav`, `main`, `footer` 등을 적재적소에 써야 검색 엔진이 우리 사이트를 잘 찾아줌.
- **Props의 중요성**: 컴포넌트를 분리할수록 Props를 잘 다루는 게 실력임. 데이터가 어디서 어디로 흐르는지 파악하는 게 핵심임.