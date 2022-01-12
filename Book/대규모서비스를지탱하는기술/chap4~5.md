### OS 캐시의 활용
- 대량의 데이터를 저장하려는 테이블은 레코드가 가능한 작아지도록 컴팩트하게 설계해야한다.
- 전체 데이터량 < 물리 메모리를 유지한다. (이러한 상황이어야 전체 데이터를 캐싱할 수 있다.)

### 인덱스의 효과
- 계산량 측면에서 개선될 뿐만 아니라 디스크 접근 횟수면에서도 개선된다.
- 대규모가 되면 될수록 인덱스는 큰 효과를 가져온다.

### 복합 인덱스 사용 
```
select * from entry where url like 'http://test/%' order by timestamp
```
- url과 timestamp에 인덱스가 모두 걸려있다고 해도 한쪽의 인덱스만 사용된다.
- 양쪽 컬럼 모두 인덱스를 태우고 싶다면 (url,timestamp)를 쌍으로 복합 인덱스를 설정해야한다.

### explain 명령
- explain 명령어를 사용했을 때 extra 열에서 using where 이외에 using filesort나 using temporary와 같은 항목이 나온다면 쿼리나 인덱스를 튜닝해갈 필요가 있다.
