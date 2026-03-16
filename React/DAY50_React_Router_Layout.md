# [DAY_48] React Router: 중첩 라우팅과 레이아웃(Layout) 설계

### 📋 학습 요약
* **React Router Dom v6**를 활용한 SPA(Single Page Application) 경로 제어 학습
* `createBrowserRouter`를 이용한 객체 기반의 라우트 설정 및 중앙 관리
* **공통 레이아웃(Layout)** 적용을 통한 코드 재사용성 및 유지보수 효율성 증대
* `useNavigate` 훅을 활용한 선언적/프로그래밍적 페이지 이동 처리

---

### 📚 배운 개념 정리
* **createBrowserRouter**: 최신 React Router에서 권장하는 방식으로, JSON 객체 형태로 경로를 정의하여 가독성이 높고 관리가 용이함
* **Outlet**: 부모 라우트 컴포넌트 내에서 자식 라우트 컴포넌트가 렌더링될 위치를 지정하는 '치환자' 역할
* **중첩 라우팅 (Nested Routing)**: 특정 경로 내부에 또 다른 경로를 정의하는 기법으로, 상단바/사이드바 등 공통 UI를 유지한 채 콘텐츠 영역만 교체할 때 필수적임
* **Index Route**: 부모 경로 접속 시 기본적으로 보여줄 자식 컴포넌트를 설정 (path 없이 `index: true` 사용)
* **useNavigate**: 특정 이벤트(로그인 완료 등) 발생 시 로직 안에서 페이지를 이동시키기 위해 사용하는 Hook

---

### 🔍 코드리뷰

#### 1. 로그인 로직 및 UX 제어 (`Login.js`)
```javascript
const navigate = useNavigate();

const handleLogin = (e) => {
  e.preventDefault();

  // 하드코딩된 인증 로직이지만, 성공 시와 실패 시의 상태 처리가 명확함
  if (loginId === "admin" && password === "1234") {
    alert("로그인 성공!");
    // replace: true 옵션을 사용하여 로그인 페이지로의 '뒤로 가기' 방지 (보안 및 UX)
    navigate("/", { replace: true });
  } else {
    setIsError(true); // 에러 상태 활성화
    setLoginId("");
    setPassword("");
  }
};

// 입력 시 에러 메시지를 초기화하는 센스 있는 이벤트 핸들러
onChange={(e) => {
  setLoginId(e.target.value);
  if (isError) setIsError(false); 
}}
```

#### 2. 중앙 집중형 라우트 설정 (`router/index.jsx`)
```javascript
const router = createBrowserRouter([
  {
    path: "/",
    element: <LayoutProb />, // 헤더/푸터가 포함된 공통 레이아웃
    children: [
      { index: true, element: <Home /> }, // 기본 경로 ("/")
      { path: "/product", element: <Products /> },
      { path: "/cart", element: <Cart /> },
    ],
  },
  {
    path: "/", // 동일 경로라도 레이아웃 구조가 다른 경우(로그인/가입) 분리 적용
    element: <SimpleLayout />, 
    children: [
      { path: "/login", element: <Login /> },
      { path: "/register", element: <Register /> },
    ],
  },
]);
```

---

그 부분 정말 예리한 질문입니다! 실무에서도 가장 많이 고민하는 **'상태 기반 UI 분기'**와 **'경로 설계'**에 대한 내용이네요. 특히 비전공자분들이 라우팅을 처음 접할 때 가장 많이 겪는 시행착오이기도 합니다.

복습하실 때 도움이 되도록, 이 두 가지 핵심 포인트만 콕 집어서 '전담 학습 기록 관리자' 스타일로 명확하게 정리해 드릴게요.

### ❓ 헷갈렸던 점 (Q&A Focus)

#### **Q1. 로그인 상태에 따라 레이아웃의 버튼(로그인/로그아웃)을 어떻게 바꾸나요?**
* **A.** 핵심은 **'전역 상태(Global State)'** 또는 **'부모 상태'**를 활용하는 것입니다.
    * 레이아웃 컴포넌트 내부에서 `isLoggedIn`과 같은 상태값을 체크하여 조건부 렌더링(삼항 연산자)을 사용합니다.
    * **실무 적용 예시:**
    ```javascript
    // Header 또는 Layout 컴포넌트 내부
    {isLoggedIn ? (
      <button onClick={handleLogout}>로그아웃</button>
    ) : (
      <button onClick={() => navigate("/login")}>로그인</button>
    )}
    ```
    * 아직 전역 상태 관리(Context API, Redux 등)를 배우기 전이라면, 우선은 최상위 컴포넌트에서 상태를 선언하고 레이아웃에 props로 전달하는 방식으로 흐름을 이해하는 것이 중요합니다.

#### **Q2. 루트 경로(`/`)를 `/prob`으로 바꿨을 때 왜 에러가 났나요?**
* **A.** 두 가지 주요 원인이 있을 수 있습니다.
    1.  **기준점 상실**: 브라우저에 주소를 입력할 때 서버나 라우터는 가장 먼저 `/`를 찾습니다. 루트가 `/prob`이 되면, 사용자가 그냥 도메인(예: localhost:3000)으로 접속했을 때 보여줄 페이지가 없어 `404 Not Found`나 에러가 발생합니다.
    2.  **중첩 경로의 상대적 위치**: `createBrowserRouter`에서 부모의 `path`가 `/prob`이 되면, 그 자식들의 경로는 자동으로 `/prob/product`, `/prob/cart`가 됩니다. 기존에 `/product`로 연결했던 링크들이 모두 깨지게 되어 에러가 발생하게 됩니다.
    * **해결책**: 서비스의 메인을 `/`로 유지하거나, 만약 전체 서비스가 `/prob` 아래에 있어야 한다면 **Basenam

---

### 💼 실무 관점
1.  **레이아웃 분리**: 실무에서는 서비스 메인(GNB 포함)과 대시보드(사이드바 포함), 로그인/가입(헤더 없음) 등 페이지 성격에 따라 레이아웃을 다원화합니다. 오늘 실습한 방식이 바로 그 기초입니다.
2.  **보안**: 클라이언트 측 라우팅만으로는 실제 보안을 완성할 수 없습니다. 추후에는 `PrivateRoute` 등을 만들어 로그인되지 않은 사용자가 `/cart` 등에 접근하는 것을 컴포넌트 수준에서 차단하는 로직이 추가되어야 합니다.