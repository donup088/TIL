### 미디어쿼리사용
```
//500~600
 @media(max-width: 600px) {
            body {
                background-color: green;
            }
        }
//0~500
@media(max-width:500px) {
    body {
        background-color: red;
    }
}
//601~
@media(min-width:601px){
    body{
        background-color: blue;
    }
}
```
- 코드의 순서도 중요하다. cascading과 연관이 있다.
```
   @media(max-width:500px) {
            .content {
                flex-direction: column;
            }

            .content nav, .content aside {
                border: none;
                flex-basis: auto;
            }
            main{
                order:0;
            }
            nav{
                order:1;
            }
            aside{
                order:2;
            }
        }

```