### 가상 클래스 선택자
- :link - 방문한 적이 없는 링크
- :visited - 방문한적이 있는 링크
- :hover - 마우스를 올려놓았을 때
- :active - 마우스를 클릭했을 때
- :focus - focus가 되었을 때

### Cascading
1. style 속성
2. id selector
3. class selector
4. tag selector
- css가 중복되어 적용될 때 위의 순서에 따라 적용된다.
- !important 를 사용하면 우선순위를 높일 수 있지만 좋은 방법은 아니다.

### font
- px (사용자가 font-size 조절을 해도 바뀌지 않는다.)
- em
- rem (사용자가 font-size를 조절할 때 바뀐다.)
- font-family 로 글꼴 설정.
- font-weight 로 글자 두께 설정.
- line-height 로 줄간격 설정.

### text-align
- text-align: justify 를 사용하면 양쪽이 균등하게 분배된다.

### backgroud
- background-image: url()
- background-repeat: no-repeat (반복 여부 결정 x축 y축에 따라서도 결정 가능)
- background-size (px로 정할 수 있고 cover와 contain 속성 사용가능)
- background-position
- background-color: azure

### inline vs block
- inline 속성을 가지면 높이와 너비가 없다. 따라서 위 아래 margin을 줄 수 없다.
- block 은 margin, padding, border 를 속성으로 가질 수 있다.
