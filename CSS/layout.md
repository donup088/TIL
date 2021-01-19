### inline vs block
- inline은 한줄에 이어서 나온다.
- block은 한줄을 다차지해서 다음 속성이 다음줄로 넘어간다.

### Box Model
- margin: 엘리먼트와 엘리먼트 사이의 간격
- padding: 본문과 테두리 사이의 간격
- inline 속성은 width, height 속성을 설정해도 무시된다.

### box-sizing
- box-sizing의 기본값은 content-box;이다.
    - content-box는 width와 height를 조절할 때 content 영역으로 조절되는 것을 의미한다.
    - border-box로 바꾸면 border를 기준으로 하여 조절된다.

### margin
- 부모태그와 자식태그의 마진겹침
    - 부모엘리먼트가 시각적 효과없이 투명한 효과일 때 부모 마진과 자식마진중 큰 마진으로 적용된다.

### position
- 기본값은 static으로 위치가 변하지 않는다.
- 위치를 변하게 하고 싶다면 position을 relative 바꿔준다.(부모태그에 대하여 상대적으로 바뀐다.)
```
//부모 태그를 기준으로 왼쪽으로 100px 위에서부터 100px 떨어진다.
#me{
    position: relative;
    left: 100px;
    top:100px;
}
```
- absoulte: 가장 외곽을 둘러싼 html태그 기준으로 위치가 설정된다.
    - static 이 아닌 다른 속성을 가진 부모태그를 기준으로 위치가 설정된다.
- fixed: 스크롤에 관계없이 고정된 형태의 UI를 만들 수 있다.

- 다단 만들기
    - column-count: 컬럼의 수 
    - column-gap: 컬럼 사이 간격 
    - column-rule-style: 컬럼 사이를 나누는 스타일 
    - column-rule-width: 스타일의 두께
    - column-rule-color: 스타일의 색 
    - column-span: all -> 컬럼에 따라 나뉘지 않고 합쳐진다. ex) 제목에 사용가능
```
.column{
    text-align: justify;
    column-count: 4;
/*        column-width: 200px;*/
    column-gap:30px;
    column-rule-style: solid;
    column-rule-width: 5px;
    column-rule-color: red;
}
h1{
    column-span: all;
}
```