# [DAY_49] React와 FastAPI Full-stack 연동: CRUD 구현 및 CORS 정책 이해

### 📝 학습 요약
* **Full-stack 통신**: React(Frontend)에서 Axios 라이브러리를 이용해 FastAPI(Backend) 서버와 데이터를 주고받는 비동기 통신 구조 학습
* **REST API CRUD**: HTTP 메서드(GET, POST, PUT, DELETE)를 활용한 할 일 목록(Todo)의 생성, 조회, 수정, 삭제 기능 구현
* **CORS(Cross-Origin Resource Sharing)**: 서로 다른 도메인(출처) 간의 자원 공유를 허용하기 위한 보안 정책 및 설정 방법 습득

### 📚 배운 개념 정리
* **Axios**: 브라우저와 Node.js를 위한 Promise API 활용 HTTP 비동기 통신 라이브러리. `fetch`보다 사용이 간편하고 JSON 자동 변환 기능을 제공함.
* **useEffect (Side Effect 처리)**: 컴포넌트가 렌더링될 때 서버에서 데이터를 가져오는 등 외부 시스템과의 동기화를 위해 사용함.
* **CORS (교차 출처 자원 공유)**: 브라우저 보안 정책상 기본적으로 다른 도메인으로의 요청을 차단함. FastAPI 측에서 `CORSMiddleware` 설정을 통해 특정 오리진(예: http://localhost:3000)의 접근을 허용해야 통신이 가능함.
* **불변성 유지 (Immutability)**: React State를 업데이트할 때 `setTodos([...todos, newTodo])`와 같이 기존 배열을 복사하여 새로운 배열을 생성해야 함.

### 💻 코드리뷰 (TodoList.js)
```javascript
import { useState, useEffect } from "react";
import axios from "axios";

// 백엔드 API 서버 주소 상수로 관리 (유지보수 용이)
const API_URL = "http://localhost:8000/todos";

const TodoList = () => {
  const [todos, setTodos] = useState([]); // 서버 데이터를 저장할 상태 배열
  const [input, setInput] = useState("");   // 사용자의 입력을 관리할 상태

  // [조회] 컴포넌트 마운트 시 최초 1회 실행
  useEffect(() => {
    const fetchTodos = async () => {
      try {
        // 비동기 함수 내에서 서버 데이터를 GET 요청
        const response = await axios.get(API_URL);
        setTodos(response.data); // 성공 시 데이터 바구니에 저장
      } catch (error) {
        console.error("데이터 로드 실패:", error);
      }
    };
    fetchTodos();
  }, []);

  // [등록] 사용자가 입력한 내용을 서버에 저장하고 UI 갱신
  const handleadd = async () => {
    if (input.trim() === "") return;
    // POST 요청 시 본문(Body)에 데이터 실어서 전송
    const response = await axios.post(API_URL, { text: input, done: false });
    // 스프레드 연산자(...)를 활용해 불변성을 지키며 배열 업데이트
    setTodos([...todos, response.data]); 
    setInput(""); 
  };

  // [삭제] 특정 ID의 데이터를 서버에서 지우고 목록 재구성
  const handleDelete = async (id) => {
    await axios.delete(`${API_URL}/${id}`);
    // filter를 사용해 삭제된 항목만 제외한 새로운 배열 생성
    setTodos(todos.filter((t) => t.id !== id));
  };

  // [완료] 특정 항목의 상태값(토글)을 서버에 반영
  const handleToggle = async (todo) => {
    // 백엔드에서 제공하는 토글 전용 엔드포인트 호출
    const response = await axios.put(`${API_URL}/${todo.id}/toggle`);
    // map을 이용해 변경된 항목만 교체하고 나머지는 유지
    setTodos(todos.map((t) => (t.id === todo.id ? response.data : t)));
  };

  return (
    // ... JSX 생략 (UI 렌더링 로직)
  );
};

export default TodoList;
```

### ❓ 헷갈렸던 점 (Q&A)
* **Q. 서버에 데이터는 저장되는데 화면이 안 바뀌어요!**
  * A. React는 주소값이 바뀌어야 변화를 감지합니다. `todos.push()`를 하면 주소값이 같아 렌더링되지 않습니다. 반드시 `[...todos]`를 써서 **새로운 배열**을 만들어 `setTodos`에 넣어야 합니다.
* **Q. 백엔드 서버를 켰는데 브라우저 콘솔에 'CORS error'라고 빨간 글씨가 떠요.**
  * A. 이는 브라우저의 보안 장치 때문입니다. FastAPI 코드 상단에 `CORSMiddleware`를 추가하고 `allow_origins=["*"]` 또는 특정 주소를 허용해 주어야 프론트엔드의 요청을 받아들입니다.

### 🏢 실무 관점
* **에러 핸들링**: 실무에서는 서버가 꺼져 있거나 네트워크 불안정 상황에 대비해 모든 `axios` 요청에 `try-catch`문을 사용하여 사용자에게 오류 메시지를 보여주는 처리가 필수적입니다.
* **환경 변수 관리**: `API_URL`은 로컬 개발 시와 실제 서비스 시에 주소가 다릅니다. 이를 위해 `.env` 파일을 사용하여 환경 변수로 관리하는 것이 보안 및 운영 효율성 측면에서 우수합니다.
* **Optimistic Update (낙관적 업데이트)**: 사용자 경험을 위해 서버 응답을 기다리지 않고 화면을 먼저 바꾼 뒤, 실패했을 때 되돌리는 기법을 적용하기도 합니다.