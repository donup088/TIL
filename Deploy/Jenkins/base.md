### AWS, jenkins 사용 CI,CD 구축하기
- jenkins 용 ec2 생성
- jenkins 용 ec2 접속후 java 설치
    ```
     sudo amazon-linux-extras list
     sudo amazon-linux-extras install 33
    ```
- git 설치
    ```
    sudo yum install git -y
    ```
- jenkins 설치 : jenkins.io 를 들어가서 링크를 복사하고 wget 명령어를 사용하여 다운받는다.
- docker 설치
    ```
    sudo yum install docker -y
    ```
- docker 실행
    ```
    sudo systemctl start docker
    ```
- public, private key 생성
    ```
    ssh-keygen -t rsa
    ```
- jenkins 실행 : java -jar 명령어 사용 , 실행후 키입력하여 로그인

- 애플리케이션 ec2 실행시킨 뒤 docker 설치
- jenkins 서버에서 public key 생성한 것을 애플리케이션 서버 ./ssh authorized keys에 붙여넣는다.
- 위의 방법이 잘 안되서 애플리케이션 보안그룹을 수정했다.
- jenkins에 public over ssh 설치
- jenkins에 설정을 들어가서 애플리케이션 설정
    - key path 설정
        - key path 설정이 안될 시 private key를 스크립트로 넣어도된다. --begin-- --end-- 이부분까지 다 넣어야한다.
    - Name : aws 애플리케이션 이름
    - Hostname : 애플리케이션 private 주소
    - username : ec2-user
- jenkins 서버를 하나 두고 app server를 따로 둬서 jenkins에서 빌드하고 배포는 app server로 할 수 있다.(위 과정에서 private key를 등록하고 host name을 입력한 server로 배포 가능)

- jenkins github 플러그인 설치

### jenkins github 연동
- jenkins 에 아이템 설정에 들어가서 git 아이디와 비밀번호 추가
- github에서 webhooks 추가
    - http://jenkins주소/github-webhook/
- git push를 하면 jenkins에서 빌드가 자동으로 된다.

- 파이프라인 사용
    - github에서 webhooks 추가
        - http://jenkins주소/github-webhook/
    - github Personal token 생성하고 Secret 값 얻기
    - jenkins 관리 -> 시스템설정 -> git server add -> Secret text 생성 (위에서 만든 토큰사용)
    - github repository webhook에서 Secret 설정 (token값)
    - 스크립트 설정을 하면 소드코드에 있는 Jenkinsfile 폴더를 스크립트로 사용가능하다.


### jenkins 설치,실행
1. sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo
2. sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
3.sudo yum install -y java-1.8.0-openjdk jenkins git docker
4. alternatives --config java 자바 버전 선택
5. service jenkins start
- sudo cat /var/lib/jenkins/secrets/initialAdminPassword 처음 어드민 비밀번호를 알 수 있다.
6. jenkins 포트변경
    ```
    sudo vim /etc/sysconfig/jenkins 
    ```

### jenkins credentials
- jenkins 관리 -> Manage credentials -> jenkins -> Global credentials 생성
- git과 연결하기 위해 git에서 accesstoken 생성 (profile -> settings -> developer settings -> accesstoken 생성 repo 체크)
- 만들어진 accesstoken으로 jenkins Global credentials 생성(usename:git아이디,password:accesstoken)
- AWS에서 iam 사용자 추가 (awsAccessKey,awsSecretKey) 
- iam 사용자 awsAccessKey,awsSecretKey 로 Global credentials 생성 -> jenkins가 내 AWS 리소스에 접근이 가능한 상태가 된다. 

### jenkins docker permission 에러
- sudo usermod -a -G docker jenkins 한 뒤 jenkins 재시작


### Spring boot-jenkins-docker 전체적인 흐름
1. jenkins git credential 설정
1. git과 jenkins 연동하기 webhook 사용
2. Jenkinsfile 작성 (pipeline)
    - git clone 
    - 운영DB 설정 옮겨주기(application.yml)
    - 빌드
    - 도커이미지생성
    - 도커 run



