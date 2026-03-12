# [DAY_실습] React useEffect를 활용한 동적 데이터 페칭 및 상품 목록 구현

### 📝 학습 요약
* **React Hook (`useState`, `useEffect`)**: 컴포넌트의 상태를 관리하고, 생명주기에 따른 부수 효과(Side Effect)를 제어함
* **비동기 데이터 통신 (`async/await`, `fetch`)**: 외부 API(DummyJSON)로부터 데이터를 비동기적으로 가져와 화면에 렌더링함
* **조건부 렌더링**: 로딩 상태(`loading`)에 따라 사용자에게 적절한 UI(로딩 메시지 vs 상품 목록)를 제공함
* **의존성 배열 (Dependency Array)**: 특정 상태값이 변경될 때만 코드가 실행되도록 최적화하는 방법을 학습함

---

### 📚 배운 개념 정리
#### 1. useEffect (자동 비서 Hook)
* **개념**: 컴포넌트가 화면에 나타날 때(Mount)나 특정 데이터가 변할 때(Update) 자동으로 실행되는 함수임
* **역할**: API 호출, 타이머 설정, 구독 등 '함수 외부의 데이터'를 다루는 작업을 수행함
* **의존성 배열**: `useEffect(함수, [값1, 값2])`에서 배열 안의 값이 바뀔 때만 함수를 재실행함. 빈 배열 `[]`을 넣으면 처음 한 번만 실행됨

#### 2. 비동기 처리 (async/await)
* **개념**: 데이터를 서버에서 가져오는 동안 프로그램이 멈추지 않고 다른 일을 할 수 있게 하는 방식임
* **try-catch-finally**: 
    - `try`: 정상적인 로직 실행
    - `catch`: 에러 발생 시 처리
    - `finally`: 성공/실패 여부와 상관없이 마지막에 반드시 실행 (예: 로딩 종료)

#### 3. 동적 쿼리 스트링 (Query String)
* **개념**: URL 주소 뒤에 `?sortBy=price&order=asc`와 같은 형식으로 조건을 붙여 서버에 전송함
* **활용**: 사용자가 클릭한 버튼에 따라 서버에서 '정렬된 결과'만 쏙 골라 받아올 수 있음

---

### 🔍 코드리뷰
```javascript
import React, { useState, useEffect } from "react";

const ProductList = () => {
  // [상태 관리] 상품 데이터, 정렬 기준, 정렬 순서, 로딩 상태를 각각 관리
  const [products, setProducts] = useState([]);
  const [order, setOrder] = useState("asc");
  const [sortBy, setSortBy] = useState("id");
  const [loading, setLoading] = useState(false);

  // [설계 의도] API 호출 로직을 별도 함수로 분리하여 재사용성 높임
  const fetchSortedProducts = async () => {
    try {
      setLoading(true); // 데이터 요청 전 로딩 시작
      // 템플릿 리터럴을 활용해 상태값(sortBy, order)을 URL에 동적으로 주입
      const response = await fetch(
        `https://dummyjson.com/products?sortBy=${sortBy}&order=${order}&limit=30`,
      );
      const data = await response.json();
      setProducts(data.products); // 받아온 데이터를 상태에 저장 -> 리렌더링 발생
    } catch (error) {
      console.error("데이터 통신 에러:", error);
    } finally {
      setLoading(false); // 요청 완료 후 로딩 종료
    }
  };

  // [중요] 의존성 배열에 [order, sortBy] 주입
  // 사용자가 정렬 버튼을 눌러 상태가 바뀌면 이 Effect가 감지하여 fetchSortedProducts를 재실행함
  useEffect(() => {
    fetchSortedProducts();
  }, [order, sortBy]);

  return (
    <div className="p-4">
      {/* 중략: 버튼 클릭 시 setSortBy와 setOrder를 업데이트하여 useEffect 실행 유도 */}
      {/* 조건부 렌더링: loading 상태에 따라 삼항 연산자로 UI 전환 */}
      {loading ? (
        <div className="text-center">로딩 중...</div>
      ) : (
        <div className="flex flex-wrap gap-4">
          {products.map((product) => (
            <div key={product.id} className="border p-4 rounded-xl">
              {/* key값은 React가 리스트 항목을 효율적으로 추적하기 위한 필수 요소임 */}
              <h3 className="font-bold truncate">{product.title}</h3>
              <p>가격: ${product.price}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

---

### ❓ 헷갈렸던 점 (Q&A)
**Q. useEffect의 의존성 배열을 비워두면(`[]`) 어떻게 되나요?**
**A.** 컴포넌트가 처음 화면에 그려질 때(Mount) 딱 한 번만 실행됩니다. 만약 위 코드에서 배열을 비워두면, 사용자가 '가격순' 버튼을 눌러도 서버에서 새로운 데이터를 가져오지 못해 화면이 변하지 않습니다.

**Q. 왜 로딩 상태(`loading`)를 사용해야 하나요?**
**A.** 네트워크 통신은 시간이 걸리는 작업입니다. 데이터가 오기 전까지 `products` 배열은 비어있으므로 사용자에게 빈 화면 대신 "일하고 있다"는 신호를 주는 것이 UX(사용자 경험) 측면에서 매우 중요합니다.

---

### 💼 실무 관점
1.  **Race Condition 방지**: 실무에서는 이전 요청이 끝나기 전에 새로운 요청이 들어오면 이전 것을 취소하는 로직(`AbortController`)을 추가하여 데이터 꼬임을 방지함
2.  **에러 핸들링**: 단순히 `console.error`로 끝내지 않고, 사용자에게 "네트워크가 불안정합니다" 같은 알림창(Toast)을 띄워주는 것이 기본임
3.  **데이터 캐싱**: 매번 서버에 묻기보다는 한 번 가져온 데이터는 일정 시간 보관하는 `React Query` 같은 라이브러리를 현업에서 선호함