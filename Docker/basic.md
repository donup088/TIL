### 도커 이미지와 컨테이너
- 도커 이미지는 프로그램을 실행하는데 필요한 설정이나 종속성을 가지고 있으면 도커 이미지를 이용해서 컨테이너를 생성하며 도커 컨테이너를 이용하여 프로그램을 실행시킨다.

### 도커 작동 순서
1. 도커 클라이언트에 명령어 입력 후 도커 서버로 보냄
2. 도커 서버에서 컨테이너를 위한 이미지가 이미 캐쉬가 되어 있는지 확인
3. 없으면 도커 허브에서 다운 받고 있다면 가지고 있는 이미지로 컨테이너 실행

### 도커 명령어 사용
- docker run -i -t ubuntu:14.04
    - -i -t 옵션은 컨테이너와 상호 입출력을 가능하게한다. -i -t 옵션을 -it로 줄일 수 있다.
    - docker run 명령어는 도커 이미지를 pull한 뒤에 컨테이너를 생성하고 컨테이너를 실행시키고 안으로 들어간다.
    - pull create start 명령어를 일괄적으로 함
- docker start : 도커 실행 -a 옵션을 주면 attach도 함께 실행가능
- docker attach 컨테이너이름 : 컨테이너 내부로 들어감
- docker stop : 하던 작업들을 완료하고 컨테이너를 중지 시킨다.
- docker kill : 바로 컨테이너를 중지 시킨다.
- docker ps : 컨테이너 목록 확인(실행중) -a 옵션을 주면 정지된 것도 볼 수 있다.
- docker rm : 컨테이너 삭제 (-f 옵션 사용하면 실행중이어도 삭제가능)
- docker port : 호스트와 연결된 포트 확인
- docker run vs docker exec
    - run -> 새로운 컨테이너를 만들어서 실행
    - exec -> 이미 실행중인 컨테이너에 명령어 전달
- docker exec -it 969a3 sh 를 사용하면 컨테이너안에 쉘환경으로 접근가능하여 명령어를 계속 사용할 수 있다. 나올 때는 ctrl+d 를 누르면된다.
### docker --link 옵션
- docker 컨테이너는 재시작될 때마다 내부 IP가 바뀐다.
- --link 옵션은 내부 IP를 알 필요 없이 항상 컨테이너에 별명으로 접근하도록 설정한다.

### 도커 이미지 생성
1. dockerfile 생성
    - 도커 이미지를 만들기 위한 설정 파일
    - 생성과정
        1. 베이스 이미지(OS) 명시
        2. 추가적으로 필요한 파일을 다운 받기 위한 명령어
        3. 컨테이너 시작시 실행될 명령어
        ```
        //베이스 이미지 명시
        FROM baseImage

        //추가적으로 필요한 파일 다운로드
        RUN command

        //컨테이너 시작시 실행될 명령어
        CMD ["executable"]
        ```
2. 도커 클라이언트에 전달
    - Dockerfile이 있는 폴더에서 docker build ./ 명령어를 사용한다.
        - 도커 파일에 입력된 것들이 클라이언트에 전달되어 도커 서버가 인식하게 해야한다.
    - 베이스 이미지에서는 다른 종속성이나 새로운 커맨드를 추가할 때는 임시 컨테이너를 만든 후 그 컨테이너를 토대로 새로운 이미지를 만든다.그리고 임시 컨테이너는 지워준다.
    - docker build -t donup08/hello:latest ./   
        - 이미지에 태그를 주고 기억하기 쉬운 이름을 줄 수 있다.
3. 도커 서버에서 작업
4. 이미지 생성

### docker volume 사용
- volume 공유를 통해 데이터 저장을 할 수 있다.
- 컨테이너가 아닌 외부에서 데이터를 저장할 수 있다.
- volume 생성
```
docker volume create --name myvolume
```
- 생성된 volume 을 사용하는 컨테이너 생성
    - myvolume_1과 myvolume_2는 myvolume을 공유하여 /root에 저장된다.
    ```
    docker run -i -t --name myvolume_1 \-v myvolume:/root/ \ubuntu:14.04
    ```
    ```
    docker run -i -t --name myvolume_2 \-v myvolume:/root/ \ubuntu:14.04
    ```
- 사용되지 않는 docker volume 모두 삭제
    ```
    docker volume prune
    ```


### docker compose 사용
- 멀티 컨테이너 상황에서 쉽게 네트워크를 연결시켜주기 위해서 사용한다.
- docker-compose 다운로드
    ```
    sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```
- docker-compose.yml 파일 생성
    ```
    version: "3"    // 도커 컴포즈의 버전
    services:       // 실행하려는 컨테이너 정의
        redis-server:   //컨테이너 이름
            image: "redis"  //컨테이너에서 사용하는 이미지
        node-app:   //컨테이너 이름
            build: .    // 현 디렉토리에 있는 Dockerfile 사용
            ports:      //포트 매핑
            - "5000:8080"
    ```
    ```
    version: "3"
    services: 
    react:
        build: 
            context: .
            dockerfile: Dockerfile.dev
        ports: 
        
        volumes: 
            - /usr/src/app/node_modules
            - ./:usr/src/app
        stdin_open: true
    ```
- docker-compose up 사용 --build 옵션 추가가능
- docker-compose down : 앱 작동 중지


### Travis.ci와 docker 사용
- .travis.ci 생성
    - access_key_id와
    ```
    sudo: required //관리자 권한 찾기

    language: generic

    services:
        - docker //도커 환경구성

    before_install: 
        - echo "start Creating an image with dockerfile"
        - docker build -t donup08/docker-react-app -f Dockerfile.dev .

    script:
        - docker run -e CI=true donup08/docker-react-app npm run test -- --coverage

    deploy:
        provider: elasticbeanstalk
        region: "ap-northeast-2"
        app: "docker-react-app"
        env: "Dockerreactapp-env"
        //elasticbeanstalk을 만들면 s3 버킷이 자동으로 생성된다.
        bucket_name: "elasticbeanstalk-ap-northeast-2-978211270598" 
        bucket_path: "docker-react-app"
        on:
            branch: master
        // AWS에서 사용자 생성해야함
        access_key_id: $AWS_ACCESS_KEY
        secret_access_key: $AWS_SECRET_ACCESS_KEY
    ```