## ec2 프리티어 사용시 jenkins 서버 swap 메모리 사용
- t2.micro를 사용하면 jenkins 서버 메모리 부족으로 서버가 멈추는 현상이 종종 발생한다.
- swap 메모리를 사용하여 이를 해결한다.
    ```
    sudo dd if=/dev/zero of=/swapfile bs=128M count=16

    sudo chmod 600 /swapfile

    sudo mkswap /swapfile

    sudo swapon /swapfile

    sudo swapon -s

    sudo vi /etc/fstab 마지막줄에 /swapfile swap swap defaults 0 0 추가
    ```
- 메모리 확인
    ```
    free -h
    ```


## spring boot 서버 jenkins, CodeDeploy 사용으로 배포 자동화 시키기

### Jenkins 서버 java, git, jenkins 설치
- java 설치
```
sudo amazon-linux-extras list
sudo amazon-linux-extras install 33
```
- git 설치
```
sudo yum install git -y
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

## react 로 구성된 frontend 배포 자동화 시키기

### jenkins 설정
- NodeJs 플러그인 설치 후 global configure 에서 nodejs 버전 선택
- git 연결은 위와 같은 방법으로한다.
- 빌드 환경 Provide Node & npm bin/ folder to PATH 선택
- execute shell 작성
    ```
    cd frontend
    npm install
    npm run build
    ```
- s3 버킷생성, codedeploy 배포그룹 생성
- jenkins에서 codedeploy 설정
    - include files에서 build/**/**/*,build/*, appspec.yml, scripts/* 추가
        - /build/static/css , /build/static/js 파일도 포함하기 위해 build/**/**/* 을 추가하였다.

### nginx 설정
```
server {
    listen          80;
    server_name     #도메인 or IP 주소;

    root /var/www/html/dist;  #Vue build 결과물이 있는 경로 (서버안쪽으로 경로를 잡아줘야한다.)
    index   index.html      index.htm;  

    # ELB 사용시 http 요청을 https 로 redirect 해주기 위함
    if ($http_x_forwarded_proto = 'http'){     
        return 301 https://도메인 or IP 주소$request_uri;
     }

    # 봇 차단
    if ($http_user_agent = "") {
        return 403;
    }

    # 봇 차단
    if ($http_user_agent ~* (MJ12bot|AhrefsBot|SemrushBot|ltx71) ) {
        return 403;
    }

    # vue router history 모드 사용시 해줘야함
    location / {
            try_files $uri $uri/ /index.html;
    }

    # api 요청 라우팅
    location /api {
            proxy_pass http://IP주소;
    }
}
```