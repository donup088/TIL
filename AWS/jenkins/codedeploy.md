## spring boot 서버 jenkins, CodeDeploy 사용으로 배포 자동화 시키기

### Jenkins 서버 java, git, jenkins 설치
- java 설치
```
sudo amazon-linux-extras list
sudo amazon-linux-extras install 33
```
- git 설치
```
sudo yum git install -y
```
- jenkins 설치
```
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum -y install jenkins
```

### jenkins 실행
```
sudo systemctl start jenkins
```
- 첫 비밀번호 확인
    - sudo cat /var/lib/jenkins/secrets/initialAdminPassword 처음 어드민 비밀번호를 알 수 있다.
- jenkins 포트변경
    ```
    sudo vim /etc/sysconfig/jenkins 
    ```
- 플러그인 설치
    - AWS CodeDeploy

### 배포전 Aws 설정
- app server 역할 생성 후 ec2 인스턴스에 적용
    - AmazonS3FullAccess
    - AWSCodeDeployFullAccess

### AppServer 설정
- java 설치, Code Deploy Agent 설치
    ```
    yum install -y ruby
    curl -O https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install
    chmod +x ./install
    ./install auto
    ```

### ELB 생성, CodeDeploy 생성
- ELB 생성시 주의할점 : ec2 보안그룹과 elb 보안그룹을 맞춰준다.
- CodeDeploy 배포 구성 생성 : ec2/온프레미스, 숫자 선택, 값 1
- CodeDeploy 애플리케이션 생성, 배포그룹 생성 : 배포 그룹 생성시 배포 구성을 위에서 만든 배포 구성을 선택한다.

### appspec.yml 파일 생성
```
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/app/backend/zip/
    overwrite: yes

permissions:
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

### s3 생성 및 IAM 사용자 생성
- 사용자 생성 후 엑세스키, 비밀키 발급
    - AmazonS3FullAccess
    - AWSCodeDeployFullAccess

### jenkins 설정
- Freestyle project 선택
- 소스 코드 관리 git 선택 후 credentials 설정 추가
- 빌드 유발 Github hook trigger 선택
- build execute shell 추가
    ```
    cd backend
    chmod +x ./gradlew
    ./gradlew build
    ```
- build 결과물 jar 파일을 옮겨주는 shell 추가
    ```
    cd backend/build/libs/
    mv *.jar ~/workspace/mini-jenkins-practice2/backend
    ```
- 빌드 후 조치 CodeDeploy 추가
    - AWS에서 만든 CodeDeploy 설정들 입력
    - Subdirectory가 있다면 입력 ex) backend
    - include files에 *.jar, appspec.yml, scripts/* 추가
    - use access/secret keys 선택후 위에서 만든 사용자 엑세스키, 비밀키 입력

### github hook 설정
- repository의 settings에 들어가서 Webhooks 클릭
- add Webhook
    - url http://jenkins주소/github-webhook/
    - content type application/json