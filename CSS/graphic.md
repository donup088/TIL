### filter
```
img{
    transition: all 1s;
}
img:hover{
    -webkit-filter: grayscale(50%);
}
h1{
    filter: blur(1px);
}
```
- filter의 사용으로 여러가지 효과를 나타낼 수 있다.
- ex) 흑백처리, 흐리게만드는 처리 등등

### transform
- transform: scale(0.5,0.5) 이처럼 사용할 수 있다.
- transform에 대하여 사용할 때 codepen을 참고하자.

### transition
- 속성 전환이 일어날 때 보다 자연스럽게 나타낼 수 있도록 한다.
- 사용할 때 display가 inline이면 inline-block으로 바꿔준다.
- UI적으로 바뀌는부분을 자연스럽게 하기 위해 사용할 수 있다.
```
a {
    font-size: 3rem;
    display: inline-block;
    /* transition-property: all;
    transition-duration: 0.1s; */
    transition: all 0.1s;
    transition: font-size 1s, transform 0.1s;
}

a:active {
    transform: translate(20px,20px);
    font-size: 2rem;
}
```

