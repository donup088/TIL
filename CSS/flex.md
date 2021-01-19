### flex
- flex를 사용하기 위해서는 부모태그와 자식태그가 존재해야한다.
- 부모 태그에 display: flex를 주게되면 기본 값으로 flex-direction: row가 설정된다. (container에게 주는 속성)


### flex-grow & flex-shrink
```
<html>
<head>
    <style>
        .contaioner{
            background-color: powderblue;
            height: 200px;
            display: flex;
            flex-direction: row;
        }
        .item{
            background-color: tomato;
            color:white;
            border:1px solid white;
            flex-grow:1;
        }
          .item:nth-child(1){
            flex-basis: 300px;
        }
        //2번째 자식태그 선택자
        .item:nth-child(2){
            flex-basis: 600px; 
            flex-shrink: 0;
        }
    </style>
</head>
<body>
    <div class="contaioner">
        <div class="item">1</div>
        <div class="item">2</div>
        <div class="item">3</div>
    </div>
</body>
</html>
```
- flex-basis로 아이템이 차지하는 면적을 정할 수 있다.
- flex-grow:1로 주게 되면 container의 자식 태그 item들이 모두 공평한 여백을 갖으며 한줄을 다 채우게 된다.
- flex-shrink:0을 하게 되면 화면이 줄어들어도 차지하고 있는 비율이 줄어들지 않는다. ex) 위의 코드에서 화면을 줄이게 되면 2번째 자식은 면적이 줄어들지 않고 1번째 자식을 면적이 줄어들게 된다.
```
flex-shrink: 1
flex-shrink: 2
두개를 비교한다면 화면이 줄어들었을 때 자신의 면적이 줄어드는 비율을 나타낸다. 2가 1보다 2배 더 줄어든다.
``` 
### flex -holy grail layout
```
<html>
<head>
    <style>
        .container {
            display: flex;
            flex-direction: column;
        }
        header {
            border-bottom: 1px solid gray;
            padding-left: 20px;
        }
        footer {
            border-top: 1px solid gray;
            padding:20px;
            text-align: center;
        }
        .content {
            display: flex;
        }
        .content nav {
            border-right: 1px solid gray;
        }
        .content aside {
            border-left: 1px solid gray;
        }
        nav, aside{
            flex-basis: 150px;
            flex-shrink: 0;
        }
        main{
            padding:10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>Css</h1>
        </header>
        <section class="content">
            <nav>
                <ul>
                    <li>html</li>
                    <li>css</li>
                    <li>js</li>
                </ul>
            </nav>
            <main>
                css 스터디
            </main>
            <aside>
                AD
            </aside>
        </section>
        <footer>
            <a href="#">홈페이지</a>
        </footer>
    </div>
</body>
</html>
```
### flex 와 관련된 속성
- align-items: center
- justify-content: center
- order:1 이런식으로 순서 정할 수 있음.
