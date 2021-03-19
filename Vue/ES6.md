### ES5 특징
- { } 에 상관없이 스코프가 설정됨
```
var sum=0;
for (var i =1;i <= 5; i++){
    sum=sum+i;
}
console.log(i); //6
```
- Hoisting
    - 함수가 아래 선언되어 있어도 위에서 사용할 수 있다.
    - 함수가 사용하는 부분보다 아래에서 재정의되어도 적용된다.
## ES6
---
### const, let
- 블록 단위 { } 로 변수의 범위가 제한
- const: 한번 선언한 값에 대해서 변경 x, 객체나 배열의 내부는 변경할 수 있다.
- let: 한번 선언한 값에 대해 다시 선언할 수 없음
    ```
    let sum=0;
    for (var i =1;i <= 5; i++){
        sum=sum+i;
    }
    console.log(i); //i is not defined
    ```
### 화살표 함수
- function 키워드를 사용하지 않고 => 로 대체
```
var sum=function(a,b){
    return a+b;
}
var sum=(a,b)=>{
    return a+b;
}
sum(10,20);
```

### 향상된 객체 리터럴
```
var dictionary={
    lookup:function(){
        console.log('found sth');
    }
};
var dictionary1={
    lookup(){
        console.log('found sth');
    }
};
```