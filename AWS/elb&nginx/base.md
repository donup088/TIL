### elb와 nginx 사용 - aws linux2
- 전체 흐름
    1. ec2 생성 후 java, docker, nginx, npm 설치
    2. elb 생성 후 ec2와 연결(elb 보안그룹을 ec2의 보안그룹 인바운드 규칙에 추가하여 elb를 통해서만 접속할 수 있도록 해야한다.)
    3. 도메인과 elb 연결 (Route 53)
    4. nginx 라우팅 설정
    5. 도메인 https 적용 (AWS Certificate Manager)
    6. nginx https 라우팅 설정

    - ec2 nginx letencrypt 방법도 가능(https 적용)


- nginx 
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

