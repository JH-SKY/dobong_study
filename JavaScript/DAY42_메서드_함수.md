### 1. 학습 요약
자바에서 실제 값이 아닌 데이터의 주소를 저장하는 '참조 자료형'의 원리를 이해하고, 배열 데이터를 효율적으로 가공하는 '고차 메서드' 활용법을 학습함.

### 2. 배운 개념 정리
* **참조 자료형 (Reference Type)**:
    * 기본 자료형(int, double 등)이 상자에 값을 직접 넣는 것이라면, 참조 자료형은 **"진짜 데이터가 있는 창고의 주소(약도)"**를 상자에 넣는 것과 같습니다.
    * **Stack(스택)** 영역에는 주소값이, **Heap(힙)** 영역에는 실제 데이터가 저장됩니다.
* **배열 고차 메서드 (Stream API)**:
    * 배열이나 리스트의 요소를 하나씩 꺼내서 "함수"를 적용해 처리하는 방식입니다.
    * `filter`: 조건에 맞는 데이터만 골라내기 (예: 짝수만 찾기)
    * `map`: 데이터를 다른 형태로 변환하기 (예: 숫자를 문자로 바꾸기)
    * `forEach`: 하나씩 꺼내서 출력하거나 작업하기

### 3. 코드리뷰
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class StudyLog {
    public static void main(String[] args) {
        // 1. 참조 자료형: 배열 선언
        // names 변수(Stack)는 실제 배열 객체(Heap)의 주소를 가리킵니다.
        String[] names = {"Kim", "Lee", "Park", "Choi"};

        // 2. 고차 메서드 활용 (Stream API)
        // 실무에서 데이터를 필터링하고 변환할 때 가장 많이 쓰는 구조입니다.
        List<String> filteredNames = Arrays.stream(names)
            .filter(name -> name.startsWith("K")) // 'K'로 시작하는 사람만 필터링
            .map(name -> name + "님")              // 이름 뒤에 "님"을 붙여서 변환
            .collect(Collectors.toList());        // 최종 결과를 리스트로 저장

        // 결과 출력
        filteredNames.forEach(System.out::println); 
    }
}
/*
[설계 의도]
- Arrays.stream(): 배열을 흐르는 강물(Stream)처럼 만들어 연속 처리가 가능하게 함.
- 가독성: 반복문(for)과 조건문(if)을 길게 쓰는 대신, 메서드 체이닝을 통해 '무엇을 할지' 명확히 보여줌.
*/
```

### 4. 헷갈렸던 점
* **Q: `String`은 참조형인데 왜 `==` 대신 `.equals()`를 쓰나요?**
    * **A**: `==`는 Stack에 있는 **주소값**을 비교하기 때문입니다. 주소가 달라도 안의 내용(문자열)은 같을 수 있으므로, 실제 값을 비교하는 `.equals()`를 써야 정확합니다.

### 5. 실무 관점
* **왜 쓰는가?**: 실제 회사 업무에서는 수만 개의 사용자 데이터를 다룹니다. `for`문으로 하나하나 돌리는 것보다 고차 메서드를 쓰면 코드가 훨씬 짧아지고 실수를 줄일 수 있습니다.
* **주의점**: 고차 메서드(Stream)는 내부적으로 복잡한 과정을 거치므로, 아주 단순한 작업이나 극도로 빠른 성능이 필요한 곳에서는 일반 `for`문이 더 유리할 수도 있습니다. 상황에 맞는 선택이 중요합니다.