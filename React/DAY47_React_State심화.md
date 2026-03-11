# [DAY_47] React State 관리: 객체/배열 업데이트 및 State 끌어올리기

## 1. 학습 요약
* **React State 심화**: 원시 타입이 아닌 객체와 배열 타입의 상태를 안전하게 변경하는 방법 학습
* **상태 관리 패턴**: 데이터 흐름을 제어하기 위한 'State 끌어올리기(Lifting State Up)' 적용
* **컴포넌트 구조화**: 단일 책임 원칙에 따라 부모, 입력(Input), 출력(List) 컴포넌트로 분리 실습

---

## 2. 배운 개념 정리

### 1) 객체 및 배열 상태의 '불변성(Immutability)'
* **개념**: React는 상태값이 변경되었는지 판단할 때 '참조값(메모리 주소)'을 비교함
* **주의**: `state.push()`나 `state.name = 'new'`와 같이 직접 수정하면 참조값이 변하지 않아 화면이 리렌더링되지 않음
* **해결**: 전개 연산자(`...`)를 사용하여 기존 값을 복사한 **새로운 객체/배열**을 생성하여 `setState`에 전달해야 함

### 2) State 끌어올리기 (Lifting State Up)
* **개념**: 여러 하위 컴포넌트가 동일한 데이터를 공유해야 할 때, 해당 상태를 공통 부모 컴포넌트로 이동시키는 것
* **목적**: '데이터의 단일 소스(Single Source of Truth)'를 유지하여 데이터 불일치 버그 방지

### 3) 컴포넌트 분리와 Props 전달
* **부모(Container)**: 상태(State)와 상태 변경 함수(Handler)를 정의
* **자식(Presenter)**: 부모로부터 전달받은 Props를 통해 화면을 그리거나 이벤트를 발생시킴

---

## 3. 코드리뷰 (실무 예시: 사용자 관리 리스트)

```javascript
import React, { useState } from 'react';

// [부모 컴포넌트] 상태 관리의 중심
const UserManagement = () => {
  const [users, setUsers] = useState([
    { id: 1, name: '도봉이', active: true },
  ]);

  // 추가: 불변성을 지키기 위해 새로운 배열 생성
  const addUser = (name) => {
    const newUser = { id: Date.now(), name, active: true };
    setUsers([...users, newUser]); // 전개 연산자 사용
  };

  // 삭제: filter를 사용하여 조건에 맞는 요소만 남긴 새 배열 반환
  const deleteUser = (id) => {
    setUsers(users.filter(user => user.id !== id));
  };

  // 수정: map을 사용하여 특정 요소만 변경된 새 배열 반환
  const toggleUser = (id) => {
    setUsers(users.map(user => 
      user.id === id ? { ...user, active: !user.active } : user
    ));
  };

  return (
    <div>
      <h1>사용자 관리</h1>
      {/* State 끌어올리기: 부모의 함수를 자식에게 전달 */}
      <UserInput onAdd={addUser} />
      <UserList users={users} onDelete={deleteUser} onToggle={toggleUser} />
    </div>
  );
};

// [자식: 입력]
const UserInput = ({ onAdd }) => {
  const [text, setText] = useState('');
  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => { onAdd(text); setText(''); }}>추가</button>
    </div>
  );
};

// [자식: 출력]
const UserList = ({ users, onDelete, onToggle }) => (
  <ul>
    {users.map(user => (
      <li key={user.id} style={{ color: user.active ? 'black' : 'gray' }}>
        <span onClick={() => onToggle(user.id)}>{user.name}</span>
        <button onClick={() => onDelete(user.id)}>삭제</button>
      </li>
    ))}
  </ul>
);

export default UserManagement;
```

---

## 4. 헷갈렸던 점 (Q&A)

**Q. 배열에 요소를 추가할 때 왜 `push` 대신 `concat`이나 `...`을 쓰나요?**
* **A**: `push`는 기존 배열 자체를 수정하고 반환값은 배열의 길이를 리턴합니다. 반면 `concat`이나 `...`은 기존 배열은 건드리지 않고 새로운 배열을 만들어 반환하므로 React가 "어? 데이터가 바뀌었네!"라고 인식하여 화면을 새로 그려줍니다.

**Q. 부모의 함수를 자식에게 줄 때 왜 화살표 함수를 쓰나요?**
* **A**: `onDelete={deleteUser(user.id)}`라고 적으면 렌더링 시점에 함수가 즉시 실행되어 버립니다. 클릭했을 때 실행되게 하려면 `() => deleteUser(user.id)` 형태로 감싸서 전달해야 합니다.

---

## 5. 실무 관점
* **성능 최적화**: State 끌어올리기를 과하게 하면 부모가 리렌더링될 때 모든 자식이 다시 그려지는 문제가 발생할 수 있습니다. 훗날 배울 `React.memo`나 `Context API`를 통해 이를 보완합니다.
* **불변성 라이브러리**: 실무에서 관리할 객체의 깊이가 깊어지면(객체 안의 객체 등) 전개 연산자가 복잡해집니다. 이 경우 `Immer` 같은 라이브러리를 사용하여 가독성을 높이기도 합니다.