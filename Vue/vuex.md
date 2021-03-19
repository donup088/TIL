### Vuex로 해결할 수 있는 문제
1. MVC 패턴에서 발생하는 구조적 오류
2. 컴포넌트 간 데이터 전달 명시
3. 여러 개의 컴포넌트에서 같은 데이터를 업데이트 할 때 동기화 문제
- Vuex 설치
```
npm install vuex --save
```
- 기술 요소
    - state: 여러 컴포넌트에 공유되는 데이터 data
    - getters: 연산된 state 값을 접근하는 속성 computed
    - mutations: state 값을 변경하는 이벤트 로직,메서드 methods
    - actions: 비동기 처리 로직을 선언하는 메서드 aysnc methods

- mutations 사용
    - state 값을 변경할 수 있는 유일한 방법이자 메서드
    - mutation은 commit()으로 동작시킨다.
    - 예시
        - 컴포넌트
        ```
        removeTodo(todoItem, index) {
                this.$store.commit('removeOneItem',{todoItem,index})
        },
        ```
        - store.js
        ```
        removeOneItem(state,payload) {
            localStorage.removeItem(payload.todoItem.item);
            state.todoItems.splice(payload.index, 1)
        },
        ```