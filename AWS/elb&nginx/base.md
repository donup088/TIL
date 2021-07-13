## elb(Elastic Load Balancing)와 nginx 사용 aws linux2(Vue,Springboot 배포)

### 전체 흐름
1. ec2 생성 후 java, docker, nginx, npm 설치
2. elb 생성 후 ec2와 연결(elb 보안그룹을 ec2의 보안그룹 인바운드 규칙에 추가하여 elb를 통해서만 접속할 수 있도록 해야한다.)
3. 도메인과 elb 연결 (Route 53)
4. nginx 라우팅 설정
5. 도메인 https 적용 (AWS Certificate Manager)
6. nginx https 라우팅 설정
7. ec2 nginx letencrypt 방법도 가능(https 적용)
- 443(https) -> ELB -> 80 port -> Nginx -> reverse proxy -> front, back

- nginx 
    - nginx 왜 사용하는가?
        - 정적 파일을 처리하는 HTTP 서버로서의 역할
        - Proxy
            - Foward Proxy(Proxy)
                - 클라이언트가 서버로 요청할 때 직접 요청하지 않고 먼저 프록시 서버를 통해 요청하는 방식이다. 
                - 서버에게 클라이언트가 누구인지 감출 수 있다.
            - Reverse Proxy
                - 클라이언트가 서버를 호출할 때 프록시 서버가 서버를 요청하여 받은 응답을 클라이언트에게 전달한다.
                - 실제 서버의 IP를 알 수 없다.
            - Proxy 기능의 장점
                - 클라이언트나 서버 모두 IP를 숨길 수 있어 보안에 도움을 준다.
                - 프록시 서버를 사용하여 캐싱 기능과 트래픽 분산으로 성능 향상을 가져올 수 있다.

    - nginx 설치
        ```
        sudo yum install nginx
        //안될 경우 log에 나오는 설명대로 실행
        ```
    - nginx 실행
        ```
        sudo sudo service nginx start
        ```
    - nginx 설정
        ```
        sudo vi /etc/nginx/nginx.conf
        ```
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

### Jenkins를 통한 배포 계획
- git push 하면 배포가 자동으로 될 수 있도록 하는 것이 목표
- jenkins git hook 연결
- jenkins ec2 연결
- pipeline
    - frontend
        - dockerfile수정(npm install, npm build, build 결과 파일 COPY)
        - docker image 생성
        - docker push 
    - backend  
        - gradlew 권한 주기
        - gradlew build
        - application.yml 운영 설정으로 수정
        - docker image 생성
        - docker push
        - docker run (컨테이너 실행)
