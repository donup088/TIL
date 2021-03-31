### 도커 이미지와 컨테이너
- 도커 이미지는 프로그램을 실행하는데 필요한 설정이나 종속성을 가지고 있으면 도커 이미지를 이용해서 컨테이너를 생성하며 도커 컨테이너를 이용하여 프로그램을 실행시킨다.

### 도커 작동 순서
1. 도커 클라이언트에 명령어 입력 후 도커 서버로 보냄
2. 도커 서버에서 컨테이너를 위한 이미지가 이미 캐쉬가 되어 있는지 확인
3. 없으면 도커 허브에서 다운 받고 있다면 가지고 있는 이미지로 컨테이너 실행

### 도커 명령어 사용
- docker run -i -t ubuntu:14.04
    - -i -t 옵션은 컨테이너와 상호 입출력을 가능하게한다.
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
### docker --link 옵션
- docker 컨테이너는 재시작될 때마다 내부 IP가 바뀐다.
- --link 옵션은 내부 IP를 알 필요 없이 항상 컨테이너에 별명으로 접근하도록 설정한다.

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
