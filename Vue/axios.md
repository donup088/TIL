### axios 설치
```
npm i axios --save
```

### api 관리 
- api 폴더를 만들어서 axios를 관리한다.
```
import axios from 'axios';

// 1. HTTP Request & Response와 관련된 기본 설정

const config={
    baseUrl: 'https://api.hnpwa.com/v0/'
}

// 2. API 함수들 정리
function fethchNewsList(){
    return axios.get(`${config.baseUrl}news/1.json`);
    // return axios.get(config.baseUrl+'news/1.json');
}

function fetchAskList(){
    return axios.get(`${config.baseUrl}ask/1.json`);
}

function fetchJobsList(){
    return axios.get(`${config.baseUrl}jobs/1.json`);
}

export{
    fethchNewsList,
    fetchAskList,
    fetchJobsList
}
```
- 사용하는 곳에서 import로 가져와서 사용한다.
```
import { fethchNewsList } from '../api/index.js';

export default {
    data() {
        return {
            users: []
        }
    },
    created() {
        var vm = this;
        fethchNewsList()
            .then(response => {
                console.log(response);
                vm.users = response.data;
            })
            .catch(error => console.log(error))
    }
}
```