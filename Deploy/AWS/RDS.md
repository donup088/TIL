# RDS (Relational Database Service)

## RDS 인스턴스 생성하기
- 표준 생성 MariaDB 선택, 퍼블릭 엑세스 기능 체크
- 운영 환경에 맞는 파라미터 설정하기
  - 파라미터 그룹 -> 파라미터 그룹 생성 -> 생성 완료 후 생성된 파라미터 그룹을 클릭해서 파라미터 편집  
  - time_zone 검색하여 Asia/Seoul 선택
  - character_set 설정 (utf8은 이모지를 저장할 수 없다.)
    * character_set_client -> utf8mb4
    * character_set_connection -> utf8mb4
    * character_set_database -> utf8mb4
    * character_set_filesystem -> utf8mb4
    * character_set_results -> utf8mb4
    * character_set_server -> utf8mb4
    * collation_connection -> utf8mb4_general_ci
    * collation_server -> utf8mb4_general_ci
  - max_connections -> 150
  - 파라미터 설정을 마쳤다면 데이터베이스에 연결(데이터베이스 -> 수정 -> DB 파라미터 그룹수정)
  - 파라미터 설정이 제대로 반영되지 않을 수도 있기 때문에 인스턴스 재부팅을 한다.
- 보안 그룹 목록 중 EC2에 사용된 보안 그룹의 그룹 ID를 복사하고 RDS 보안 그룹의 인바운드에 복사된 보안 그룹 ID와 본인의 IP를 추가한다.

## 인텔리제이 Database 플러그인 설치 및 사용
- Database Navigator 설치후 Action(DB Browser)를 들어가서 실행시킨다.
- DB Browser에 추가버튼을 클릭하고 MySQL을 선택한다.
- 자신이 생성한 RDS 정보를 등록한다. (Host = RDS 엔드 포인트)
- DB character_set,collation 설정 확인
  ```
  use 데이터베이스명
  show variables like 'c%';
  ```
  character_set_database, collation_connection 두 항목이 latin1로 되어있을 경우
  ```
  ALTER DATABASE 데이터베이스명
  CHARACTER SET ='utf8mb4'
  COLLATE = 'utf8mb4_general_ci'
  ```
  ```
  select @@time_zone,now(); //타임존 확인
  ```
- 설정을 마친 후 mysql 문법을 사용하여 데이터베이스를 사용하면 된다.
- EC2에서 RDS 접근 확인
  ```
  sudo yum install mysql
  mysql -u 계정 -p -h Host주소
  show databases;
  ```
  