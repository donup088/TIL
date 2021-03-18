## Vue Basic
---
### Vue 인스턴스 생성과 사용
```
new Vue({
    el: ,
    template: ,
    data: ,
    methos: ,
    created: ,
    watch ,
})
```
- el: 인스턴스가 그려지는 화면의 시작점
- data: 뷰 반응성이 반영된 데이터 속성
- methods: 화면의 동작과 이벤트 로직을 제어하는 매서드
- created: 뷰의 라이프 사이클과 관련된 속성
- watch: data에서 정의한 속성이 변화했을 때 추가 동작을 수행할 수 있게 정의하는 속성

### Vue 컴포넌트
- 화면의 영역을 구분하여 개발할 수 있도록 해줌. 
- 전역 컴포넌트
    ```
    <div id="app">
            <app-header></app-header>
    </div>
    Vue.component('app-header',{
                template: '<h1>Header</h1>'
    });
    ```
- 지역 컴포넌트
    ```
    new Vue({
            el: '#app',
            components:{
                'app-footer': {
                    template: '<footer>footer</footer>'
                }
            }
        })
    ```
- 컴포넌트 통신방식
    - 상위 -> 하위 : props 전달
    - 하위 -> 사위 : event 발생
- props 사용
    ```
    <div id="app">
        <app-header v-bind:propsdata="message"></app-header>
        <app-content v-bind:propsdata="num"></app-content>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        var appHeader = {
            template: '<h1>{{ propsdata }}</h1>',
            props: ['propsdata']
        }
        var appContent = {
            template: '<h1>{{ propsdata }}</h1>',
            props: ['propsdata']
        }

        new Vue({
            el: '#app',
            components: {
                'app-header': appHeader,
                'app-content': appContent
            },
            data: {
                message: 'hi',
                num: 10
            }
        })
    </script>
    ```
- event emit
    - 자식 컴포넌트에서 이벤트를 발생시키고 부모 컴포넌트에서 자식 컴포넌트에서 발생시킨 이벤트가 발생했다는 것을 알아차리고 그 때 실행시키는 메서드를 찾아 실행시킨다.
    ```
    <div id="app">
        <p>{{ num }}</p>
        <app-header v-on:pass="logText"></app-header>
        <app-content v-on:add="addNum"></app-content>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        var appHeader = {
            template: '<button v-on:click="passEvent">click me</button>',
            methods: {
                passEvent: function () {
                    this.$emit('pass');
                }
            }
        }
        var appContent = {
            template: '<button v-on:click="addNumber">add</button>',
            methods: {
                addNumber: function () {
                    this.$emit('add')
                }
            }
        }

        new Vue({
            el: '#app',
            components: {
                'app-header': appHeader,
                'app-content': appContent
            },
            methods: {
                logText: function () {
                    console.log('hi');
                },
                addNum: function () {
                    this.num++;
                    console.log(this.num);
                }
            },
            data: {
                num: 10
            }
        });
    </script>
    ```
- 같은 레벨의 컴포넌트에 데이터 넘기기
    - 부모 컴포넌트로 이벤트 emit을 하고 부모 컴포넌트에서 데이터를 받는다.
    - 전달하려는 컴포넌트에서 props를 사용하여 부모 컴포넌트의 데이터를 가져온다.

### vue 라우터
- 라우터를 만들고 Vue에 적용시킨다.
```
 var router = new VueRouter({
            //페이지의 라우팅 정보
            routes: [
                {
                    // 페이지의 url
                    path: '/login',
                    // 해당 url에서 표시될 컴포넌트
                    component: LoginComponent
                },
                {
                    path: '/main',
                    component: MainComponent
                }
            ]
        });

        new Vue({
            el: '#app',
            router: router
        });
```
- router-link 태그와 router-view 태그 사용
```
<div>
    <router-link to="/login">Login</router-link>
    <router-link to="/main">Main</router-link>
</div>
<router-view></router-view>
```

### vue 템플릿 문법
- computed 사용
    ```
    <div id="app">
        <p>{{ num }}</p>
        <p>{{ doubleNum }}</p>
    </div>

    new Vue({
            el: '#app',
            data: {
                num: 10
            },
            computed: {
                doubleNum: function () {
                    return this.num * 2;
                }
            }
        })
    ```
- data를 사용하여 id,class 설정
    ```
    <p v-bind:id="uuid" v-bind:class="name">{{ num }}</p>
    ```
- v-if, v-show
    - v-if 는 조건에 맞지 않으면 html태그가 나타나지 않는다.
    - v-show는 조건에 맞지 않으면 html태그가 남아있고 style이 none으로 바뀌어 안보이게 된다.
    ```
    <div v-if="loading">
            Loading...
    </div>
    <div v-else>
        test user has been logged in
    </div>
    <div v-show="loading">
        Loading...
    </div>
    ```
- v-on: 사용
    - 여러가지 이벤트를 사용할 수 있다.
    ```
    <button v-on:click="logText">click me</button>
    <input type="text" v-on:keyup.enter="logText">
    ```
- watch 사용
    - 간단한 것들은 computed를 사용하는 것이 더 좋다
    - 해당 데이터가 바뀔 때 마다 설정한 함수를 실행시킬 수 있다.
    ```
    new Vue({
            el: '#app',
            data: {
                num: 10
            },
            watch:{
                num: function(){
                    this.logText();
                }
            },
            methods: {
                addNum: function () {
                    this.num++;
                },
                logText: function(){
                    console.log('changed');
                }
            }
        })
    ```