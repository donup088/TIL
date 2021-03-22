### router 설치
```
npm i vue-router --save
```

### router 관리
- router 폴더를 따로 만들고 index.js에서 router를 관리하는 것이 좋다.
```
Vue.use(VueRouter);

export const router = new VueRouter({
    mode: 'history',//url # 제거
    routes: [
        {
            path: '/',
            redirect: '/news'
        },
        {
            // path: url 주소
            path: '/news',
            // component: url 주소로 갔을 때 표시될 컴포넌트
            component: NewsView,
        },
        {
            path: '/ask',
            component: AskView,
        },
        {
            path: '/jobs',
            component: JobsView,
        }
    ]
});
```

- main.js 에서 router를 import해서 가져온다.
```
import Vue from 'vue'
import App from './App.vue'
import { router } from './router/index.js';

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
  router
}).$mount('#app')

```

### router link 사용
- to를 통해서 이동할 url을 정할 수 있다.
```
<template>
  <div>
      <router-link to="/news">News</router-link>
      <router-link to="/ask">Ask</router-link>
      <router-link to="/jobs">Jobs</router-link>
  </div>
</template>
```

### 동적 route
```
routes: [
        {
            path: '/user/:id',
            component: UserView,
        }
    ]

// NewsView
 <router-link v-bind:to="`/user/${item.user}`">{{ item.user }}</router-link>
```
- 동적 route의 데이터 사용
    ```
    //UserViews
    created() {
        const userName = this.$route.params.id;
        this.$store.dispatch('FETCH_USER', userName);
    }
    ```

### 공통 컴포넌트화 
- route name을 사용하기 위해 route에서 name을 설정해주고 name을 분기점으로 데이터를 다르게 가져온다.
- v-if 와 v-else를 사용해서 화면을 다르게 처리해준다.
    ```
    created() {
        const name = this.$route.name;
        if (name === 'news') {
            this.$store.dispatch('FETCH_NEWS');
        } else if (name === 'ask') {
            this.$store.dispatch('FETCH_ASKS');
        } else if (name === 'jobs') {
            this.$store.dispatch('FETCH_JOBS');
        }
    },
    computed: {
        listItems() {
            const name = this.$route.name;
            if (name === 'news') {
                return this.$store.state.news;
            } else if (name === 'ask') {
                return this.$store.state.asks;
            } else if (name === 'jobs') {
                return this.$store.state.jobs;
            }
        }
    }
    ```