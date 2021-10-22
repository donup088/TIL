## node 사용 배포시 이슈

### react 빌드시 힙메모리 부족으로 에러 발생
- .env 파일에 설정 추가
    ```
    NODE_OPTIONS="--max-old-space-size=4096"
    GENERATE_SOURCEMAP=false
    ```
    - 빌드시 메모리를 더 할당하도록 설정해준다.

### react 배포 자동화 과정에서 jenkins 실행 과정중 env 파일 수동으로 넣는 방법
- 상황 : frontend에서 .env 파일을 gitignore 하여 배포과정에서 빌드전에 수동으로 추가해줘야하는 상황
- .env 파일 서버에 만들고 cp -r 명령어로 하려고 했지만 permission 에러가 계속 났다.
- sudo 를 명령어에 추가하는 방법을 사용해보았지만 안됨.
- 권한문제라고 생각하여 jenkins 폴더를 찾아서 jenkins 유저 권한으로 폴더를 만들어 주었다.
- jenkins node script
    ```
    cd frontend
    cp -r /var/lib/jenkins/workspace/.env /var/lib/jenkins/workspace/mini-dev2-frontend/frontend
    npm install
    npm run build
    ```
- jenkins 서버 /var/lib/jenkins/workspace 경로에 .env 파일을 만들고 유저와 그룹 권한을 모두 jenkins로 해주었더니 해결되었다.

