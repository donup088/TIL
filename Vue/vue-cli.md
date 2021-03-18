### Vue Cli 
- 프로젝트 생성
    - vue create 프로젝트 이름
- 기본 구조
    - template에 html 코드
    - script에 js 코드
    - style에 css코드
    ```
    <template>
    
    </template>

    <script>
    export default {

    }
    </script>

    <style>

    </style>
    ```
- 기본 사용
```
<template>
  <form v-on:submit.prevent="submitForm">
    <div>
      <label for="username">id: </label>
      <input id="username" type="text" v-model="username">
    </div>
    <div>
      <label for="password">pw: </label>
      <input id="password" type="password" v-model="password">
    </div>
    <button type="submit">login</button>
  </form>
</template>

<script>
import axios from 'axios'

export default {
  data: function(){
    return{
      username: '',
      password: '',
    }
  },
  methods:{
    submitForm: function(){
      console.log(this.username,this.password);
      var url='https://jsonplaceholder.typicode.com/users';
      var data={
        username: this.username,
        password: this.password
      }
      axios.post(url,data)
      .then(function(response){
        console.log(response);
      })
      .catch(function(error){
        console.log(error);
      });
    }
  }
}
</script>
```