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