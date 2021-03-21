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
- actions 사용
    - 비동기 처리 로직을 사용할 때 actions에 선언한다.
    - actions에 있는 메서드를 사용하기 위해서는 사용하는 곳에서 dispatch를 사용한다.

- 헬퍼 함수
    - mapState
    ```
    import { mapState } from 'vuex'

    computed() {
        ...mapState(['num'])
        //num(){return this.$strore.state.num;}
    }
    //store.js
    state:{
        num: 10
    }
    ```
    - mapGetters
    ```
    import { mapGetters } from 'vuex'

    //생략

    computed: {
        // todoItems(){
        //     return this.$store.getters.storedTodoItems;
        // }
        ...mapGetters({
            todoItems: 'storedTodoItems'
        })
    }
    ```
    ```
    <li v-for="(todoItem,index) in this.todoItems" v-bind:key="todoItem.item" class="shadow">
    ```
    - mapMutations
    ```
    import { mapMutations } from 'vuex'
    export default {
        methods: {
            ...mapMutations({
                clearTodo:'clearAllItem'
            })
        }
    }
    ```
### Vuex를 사용할 때 api를 호출하여 데이터 적용하기
1. api 호출하는 부분은 actions를 사용한다.
2. actions에서 state를 바꿀 수 있는 방법은 mutations을 사용하는 것이다.
3. mutations에 메서드를 추가하여 state를 바꿀 수 있도록 한다.
```
export const store = new Vuex.Store({
    state: {
        news: []
    },
    mutations: {
        SET_NEWS(state, news) {
            state.news = news;
        }
    },
    actions: {
        FETCH_NEWS(context) {
            fethchNewsList()
                .then(response => {
                    console.log(response);
                    context.commit('SET_NEWS', response.data);
                })
                .catch(error => {
                    console.log(error);
                })
        }
    }
})
```
4. state 바뀐 부분을 화면에 적용시켜준다.