# CI &CD 

- CI란 VCS 시스템에 PUSH가 되면 자동으로 테스트와 빌드가 수행되어 안정적인 배포 파일을 만드는 과정을 CI라고 한다.
- CD란 빌드 결과를 자동으로 운영 서버에 무중단 배포까지 진행되는 과정이다.

## Travis CI 연동
- 계정 -> setting -> 깃허브 저장소 활성화
- .travis.yml 파일 설정
  ```
  language: java
  jdk:
    - openjdk8
  branches:
    only:
      - master
  # Travis CI 서버의 Home
  cache:
    directories:
      - '$HOME/.m2/repository'
      - '$HOME/.gradle'
  
  script: "./gradlew clean build"
  # CI 실행 완료시 메일로 알람
  notifications:
    email:
      recipients:
        - 본인 이메일 주소
  ```
  
#### Travis CI와 AWS S3 연동
- S3란 AWS에서 제공하는 일종의 파일 서버이다. 이미지 업로드 구현한다면 S3를 이용하여 구현하는 경우가 많다.
- Travis CI가 S3로 jar를 전달한다.
- 배포할 때 사용할 CodeDeploy는 저장 기능이 없기 때문에 Travis CI가 빌드한결과물을 받아서 CodeDeploy가 가져갈 수 있도록 보관할 수 있는 공간이 ASWS S3이다.
- 연동 과정
  - AWS key 발급 (AWS서비스에 외부 서비스가 접근할 수 없기 때문에 접근 가능한 권한을 가진 key를 생성해야한다.)
    - IAM을 사용하여 Travis CI가 AWS의 S3와 CodeDeploy에 접근할 수 있도록 한다.
    - IAM 검색 -> 사용자 -> 사용자 추가 -> 사용자의 이름과 프로그래밍 방식 엑세스 선택 -> 기존 정책 직접 연결 -> s3full검색하여 권한 추가 codedeployf 검색하여 권한 추가
  - 엑세스 키 ID와 비밀 엑세스 키를 Travis CI에 등록한다. 
    - TravisCi 설정 -> AWS_ACCESS_KEY : 엑세스 키 ID , AWS_SECRET_KEY: 비밀 엑세스 키 등록
    - 여기에 등록된 값은 .travis.yml에서 $AWS_ACCESS_KEY,$AWS_SECRET_KEY로 사용가능
- S3 버킷 생성  
  - 퍼블릭 엑세스 모두 차단하여 만든다.
- Travis CI에서 빌드하여 만든 jar파일을 S3에 올릴 수 있도록 .travis.yml파일 수정
  ```
  language: java
  jdk:
  - openjdk8
  
  branches:
  only:
  - master
  # Travis CI 서버의 Home
  cache:
  directories:
  - '$HOME/.m2/repository'
  - '$HOME/.gradle'
  
  script: "./gradlew clean build"
  
  before_install:
  - chmod +x gradlew
  # CodeDeploy는 Jar파일을 인식하지 못하므로 Jar+ 기타 설정 파일들을 모아 압축한다.
  before_deploy:
    - zip -r freelec-springboot2-webservice *
    - mkdir -p deploy
    - mv freelec-springboot2-webservice.zip deploy/freelec-springboot2-webservice.zip 
  deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY
    secret_access_key: $AWS_SECRET_KEY
    bucket: freelec-springboot-webserver-build # S3 버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private으로
    # 앞에서 생성한 deploy 디렉토리 지정, 해당 위치의 파일들만 S3로 전송
    local_dir: deploy #before_deploy에서 생성한 디렉토리
    wait-until-deployed: true
  
  # CI 실행 완료시 메일로 알람
  notifications:
  email:
  recipients:
  - 본인 메일 주소
  ```

## Travis CI, AWS S3 ,CodeDeploy 연동
- EC2가 CodeDeploy를 연동 받을 수 있게 IAM 역할 추가
  - EC2에서 사용할 것이기 때문에 역할로 처리
  - 정책에서 EC2RoleForA를 검색하여 추가
  - EC2 인스턴스 설정 -> IAM 역할 바꾸기 -> 재부팅
- CodeDeploy의 요청을 받을 수 잇게 에이전트 설치
  ```
  aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install .
  --region ap-northeast-2
  chmod +x ./install 
  sudo ./install auto
  ```
  ```
  sudo service codedeploy-agent status
  ```
- CodeDeploy에서 EC2에 접근하기 위해 권한생성
  - IAM 역할 생성 -> AWS 서비스 -> CodeDeploy

## CodeDeploy 생성  
- 컴퓨팅 플랫폼에선 EC2/온프레미스 선택
- 배포 그룹 생성  -> 배포 그룹 이름 , 서비스 역할 선택(CodeDeploy용 IAM 역할) 
  - 배포 구성 : CodeDeployDefault.AllAtOnce
  - 로드 밸런싱 활성화 해제
- Travis CI, S3, CodeDeploy 연동
  ```
  mkdir ~/app/step2 && mkdir ~/app/step2/zip
  ```
  - appspec.yml 파일 생성
  ```
  version: 0.0
  os: linux
  //source는 CodeDeploy에서 전달해준 파일 중 destination으로 이동시킬 대상을 지정함.
  //이후 jar 를 실행하는 등은 destination에서 옮긴 파일들롲 진행됨.
  files:
  - source: /
    destination: /home/ec2-user/app/step2/zip/
    overwrite: yes
  ```
  - .travis.yml 파일에 CodeDeploy 내용 추가
  ```
  ...
  - provider: codedeploy
    access_key_id: $AWS_ACCESS_KEY
    secret_access_key: $AWS_SECRET_KEY
    bucket: freelec-springboot-webserver-build
    key: freelec-springboot2-webservice.zip # 빌드 파일 압축해서 전달

    bundle_type: zip # 압축 확장자
    application: freelec-springboot2-webservice # 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
    deployment_group: freelec-springboot2-webservice-group # 웹 콘솔에서 등록한 CodeDeploy 배포그룹
    region: ap-northeast-2
    wait-until-deployed: true
  ```
  - 프로젝트를 커밋하고 푸시하면 Travis CI가 자동으로 시작되고 Travis CI가 끝나면 CodeDeploy 배포가 수행된다.
  - 배포가 끝났다면 /home/ec2-user/app/step2/zip 에 파일목록들을 확인해본다.
  
## 배포 자동화 구성
- script 디렉토리를 생성하고 deploy.sh 를 생성한다.
  ```
  #!/bin/bash
  
  REPOSITORY=/home/ec2-user/app/step2
  PROJECT_NAME=freelec-springboot2-webservice
  
  echo "> Build 파일 복사"
  
  cp $REPOSITORY/zip/*.jar $REPOSITORY/
  
  echo "> 현재 구동중인 애플리케이션 pid 확인"
  
  CURRENT_PID=$(pgrep -fl freelec-springboot2-webservice | grep jar | awk '{print $1}')
  
  echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"
  
  if [ -z "$CURRENT_PID" ]; then
      echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
  else
      echo "> kill -15 $CURRENT_PID"
      kill -15 $CURRENT_PID
      sleep 5
  fi
  
  echo "> 새 어플리케이션 배포"
  
  JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)
  
  echo "> JAR Name: $JAR_NAME"
  
  echo "> $JAR_NAME 에 실행권한 추가"
  
  chmod +x $JAR_NAME
  
  echo "> $JAR_NAME 실행"
  nohup java -jar \
      -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
      -Dspring.profiles.active=real \
      $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
  ```
- .travis.yml 파일 수정
  ```
  before_deploy:
    - mkdir -p before-deploy # zip에 포함시킬 파일들을 담을 디렉토리 생성
    - cp scripts/*.sh before-deploy/
    - cp appspec.yml before-deploy/
    - cp build/libs/*.jar before-deploy/
    - cd before-deploy && zip -r before-deploy * # before-deploy로 이동후 전체 압축
    - cd ../ && mkdir -p deploy # 상위 디렉토리로 이동후 deploy 디렉토리 생성
    - mv before-deploy/before-deploy.zip deploy/freelec-springboot2-webservice.zip # deploy로 zip파일 이동
  ```
- appspec.yml 파일 수정 
  ```
  permission:
    - object: /
      pattern: "**"
      owner: ec2-user
      group: ec2-user
  
  hooks:
    ApplicationStart:
      - location: deploy.sh 
        timeout: 60
        runas: ec2-user
  ```