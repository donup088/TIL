### 양방향 매핑
- 양방향 매핑은 단방향 매핑 2개로 볼 수 있다.
- 두 객체를 양방향 매핑을 했으면 두 객체 중 하나로 외래키를 관리해야한다.
- 처음 설계할 때 단방향 매핑으로 끝내야한다. 양방향 매핑은 반대 방향으로 조회기능이 추가된 것이다.
- 단방향 매핑을 잘해놓고 양방향 매핑은 필요할 때 추가해도 된다.

### 연관관게의 주인
- 연관관계의 주인만 외래키를 관리(등록,수정)
- 주인이 아닌 쪽은 읽기만 가능
- 주인이 아니면 mappedBy 속성으로 주인을 지정
- 외래 키가 있는 곳을 주인으로 정하자.
- 일대다라면 다쪽에 외래키를 두고 주인으로 정한다.

### 양방향 매핑시 주의점
- 값을 세팅할 때 양쪽에 모두 넣어줘야한다.
    - 값을 한쪽만 넣어주면 테스트 케이스에서 오류가 날 수 있다.
    - 트랙잭션 커밋이 안된 시점에 값을 가져오면 값이 없는 경우가 생길 수 있다.
- 연관관계 편의 메소드를 생성한다.
    ```
    public void setTeam(Team team){
        this.tema = team;
        team.getMembers().add(this);
    }
    ```
- 양방향 매핑시 무한 루프 조심하자.
    - toString()
    - lombok
    - JSON 생성 라이브러리

### 다대일 [N:1] (다쪽에서 외래키 관리)
- @ManyToOne
- 다쪽에 @JoinColum으로 fk 설정
- @OneToMany (mapped By 설정)

### 일대다 [1:N] (일쪽에서 외래키 관리)
- 이런식으로 설정할 수도 있지만 다쪽에서 외래키를 관리하는 것이 더 좋다.
- 생각하지 못한 쿼리문이 나갈 수 있다.

### 일대일 [1:1]
- 주 테이블이나 대상 테이블 중에 외래 키 선택 가능
- @OneTOOne
- 외래키를 둘 곳에는 @JoinColum으로 외래키 설정
- 양방향으로 만들 경우 조회만 할 수있게 만들 부분에는 mappedBy 지정

### 다대다[N:M]
- @ManyToMany @JoinTable 사용
- 연결 테이블에 필드 추가가 안된다.
- 생각하지 못한 쿼리가 나올 수 있다.
- 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어야한다.

### 상속관계 매핑
- 조인 전략
    - @Inheritance(strategy = InheritanceType.JOINED)
    - 상속된 엔티티를 insert 하게되면 insert 쿼리가 2번 나가게 된다.
    - 조회시 join을 통해서 가져오게 된다.
    - 테이블 정규화, 저장공간 효율화
- @DiscriminatorColumn(name="DTYPE") 을 통해서 구분할 수 있는 컬럼을 만들 수 있다. 상속받은 엔티티에서는 @DiscriminatorValue("MOVIE") 를 통해 구분되는 값을 설정할 수 있다.
- 단일 테이블 전략
    - @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    - 한테이블에 모두 들어가고 DTYPE으로 구분한다.
    - 자식 엔티티 필드에는 null을 허용해야한다.
- 구현 클래스마다 테이블 전략
    - @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
    - 모든 클래스마다 테이블이 생긴다.
- 기본적으로는 조인 전략을 사용하고 프로젝트가 간단할 때는 단일 테이블 전략을 사용해도 좋다.

### @MappedSuperClass
- 공통 매핑 정보가 필요할 때 사용한다.
- 수정 날짜, 생성 날짜와 같은 필드들에 사용.
- 엔티티가 아니고 상속 받는 클래스에 매핑 정보만 제공한다.
- 추상 클래스로 사용한다.

### Arraylist 초기화시 주의점
- Builder 패턴과 같이 사용할 때 아래 코드와 같이 사용하면 값이 null이다.
```
private List<MemberClub> memberClubs=new ArrayList<>();
```
- @Builder.Default를 사용하거나 생성자를 따로 만들어서 @Builder를 사용한다.
```
@Builder.Default
private List<MemberClub> memberClubs=new ArrayList<>();
```

### Cascade 사용
```
@OneToMany(mappedBy = "club", cascade = CascadeType.ALL)
@Builder.Default
private List<MemberClub> memberClubs=new ArrayList<>();
``` 
- OneToMany에서 연관관계의 주인으로 설정한 club이 저장될 때 아래 있는 MemberClub를 add와 같은 메소드로 값을 추가하면 Club과 함께 모두 저장된다.
- 다대다 관계를 일대다 2개로 풀어서 중간테이블이 나왔을 떄 중간테이블을 자동저장시키기 위해 사용하면 편리하다.