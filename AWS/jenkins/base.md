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
    - Name : aws 애플리케이션 이름
    - Hostname : 애플리케이션 private 주소
    - username : ec2-user

- jenkins github 플러그인 설치

### jenkins github 연동

- jenkins 에 아이템 설정에 들어가서 git 아이디와 비밀번호 추가
- github에서 webhooks 추가
    - http://jenkins주소/github-webwook/
- git push를 하면 jenkins에서 빌드가 자동으로 된다.

