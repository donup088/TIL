### 프록시
- 실제 클래스를 상속 받아서 만들어진다.
- getName() -> 영속성 컨텍스트에 초기화 요청 -> DB조회 -> 실제 Entity 생성 -> target.getName()
- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체는 원본 엔티티를 상속받아서 타입 체크할 때 instance of를 사용해야한다.
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때 프록시를 초기화하면 문제가 발생한다.(LazyInitializationException)

### 지연 로딩 LAZY
```
@ManyToOne(fetch = FetchType.LAZY)
```
- 조회시 프록시로 가져와서 해당 엔티티의 메소드를 사용할 때 쿼리문이 나간다.

### 즉시 로딩 EAGER
```
@ManyToOne(fetch = FetchType.EAGER)
```
- 엔티티를 조회할 때 연관된 엔티티도 같이 조인해서 가져온다.

### 지연 로딩 vs 즉시 로딩
- 즉시 로딩을 적용하면 예상하지 못한 SQL이 작성된다.
- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
- @ManyToOne 과 @OneToOne 은 LAZY로 설정하자.
- 지연 로딩을 사용하자.

### Cascade
- Cascasde 옵션으로 부모객체가 저장될 때 자식 객체들도 같이 저장 되게 할 수 있다.
- 부모객체가 삭제될 때 자식객체도 삭제되게 할 수도 있다.
