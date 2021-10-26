# EC2서버에 프로젝트 배포하기

- EC2에 프로젝트 clone 받기
  ```
  sudo yum install git
  git --version
  mkdir ~/app && mkdir ~/app/step1
  cd ~/app/step1
  git clone 깃허브 repository clone 주소
  ```

- 테스트 수행
  ```
  git pull
  chmod +x ./gradlew //실행 권한 추가
  ./gradlew test
  ```

- 배포 스크립트 만들기
  - git clone or git pull 을 통해 새 버전의 프로젝트 받기
  - Gradle이나 Maven을 통해 프로젝트 테스트와 빌드
  - EC2서버에서 해당 프로젝트 실행 및 재실행 
  - vim ~/app/step1/deploy.sh 파일

    ```
    #!/bin/bash
    
    REPOSITORY=/home/ec2-user/app/step1
    PROJECT_NAME=freelec-springboot2-webservice
    
    cd $REPOSITORY/$PROJECT_NAME/
    
    echo "> Git pull"
    
    git pull
    
    echo "> 프로젝트 Build 시작"
    
    ./grdlew build
    
    echo "> step1 디렉토리로 이동"
    
    cd $REPOSITORY
    
    echo "> Build 파일 복사"
    
    cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/
    
    echo "> 현재 구동중인 애플리케이션 pid 확인"
    
    CURRENT_PID=$(pgrep -f ${PROJECT_NAME}.*.jar)
    //pgrep은 process id만 추출하는 명령어이다.
    //-f 옵션은 프로세스 이름으로 찾는다.
    echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"
    
    if [ -z "$CURRENT_PID" ]; then
    echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
    else
    echo "> kill -15 $CURRENT_PID"
    kill -15 $CURRENT_PID
    sleep 5
    fi
    
    echo "> 새 어플리케이션 배포"
    
    JAR_NAME=$(ls -tr $REPOSITORY/ | grep jar | tail -n 1)
    //새로 시작할 jar 파일명을 찾는다.
    //여러 jar 파일이 생기기 때문에 tail -n으로 가장 나중의 jar 파일을 변수에 저장한다.
    echo "> JAR Name: $JAR_NAME"
    
    nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
    //일반적으로 java -jar 명령어로 사용하면 터미널 접속이 끊어지면 애플리케이션도 종료된다.
    //nohup 사용으로 터미널을 종료해도 애플리케이션이 구동되도록 한다.
    ```
    ```
    chmod +x ./deploy.sh // 실행 권한 추가
    ./deploy.sh //스크립트 실행
    ```
- 외부 secret 파일 등록
  - 서버에서 직접 설정을 가지고 있도록 한다.
    ```
    vim /home/ec2-user/app/application-oauth.properties
    ```  
  - deploy.sh 파일수정
    ```
    nohup java -jar \
      -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
      $REPOSITORY/$JAR_NAME 2>&1 &
    ```
- RDS 테이블 생성 및 프로젝트 설정
  - JPA가 사용될 엔티티 테이블과 스프링 세션이 사용될 테이블 2가지 종류를 생성함. (스프링 세션 테이블은 schema-mysql.sql 파일에서 확인할 수 있다.)
  - compile("org.mariadb.jdbc:mariadb-java-client") 추가
  - application-real.properties 추가
    ```
    spring.profiles.include=oauth,real-db
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    spring.session.store-type=jdbc
    ```
  - EC2 서버에 application-real-db.properties 생성
    ```
    spring.jpa.hibernate.ddl-auto=none
    spring.datasource.url=jdbc:mariadb://rds주소:포트명(기본은 3306)/database명
    spring.datasource.username=db계정
    spring.datasource.password=db계정 비밀번호
    spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
    ```
  -  deploy.sh 파일 수정
        ```
        nohup java -jar \
        -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
        -Dspring.profiles.active=real \
        $REPOSITORY/$JAR_NAME 2>&1 &
        ```
  - deploy.sh 실행후 
    ```
    curl localhost:8080
    ```
    실행 html 코드가 보인다면 성공.
    
---

## EC2에서 소셜로그인
- ec2 보안그룹에서 8080이 열려있도록 한다.
- 퍼블릭 DNS에 :8080을 붙여서 구글 웹 콘솔에서 승인된 리디렉션 URI 추가, 네이버에 EC2 주소 등록
- 네이버의 서비스 URL은 8080포트를 제외하고 실제 도메인 주소만 입력한다.
