### jenkins git private repository와 연결시키기
- 프로젝트 운영에 들어가면서 git repository를 private으로 바꿨다.
- CI/CD로 jenkins를 사용하고 있었기 때문에 설정을 바꿔줘야했다.
- 설정을 바꾸기 위해선 repository의 setting에 접근 권한을 가지고 있어야한다.

1. jenkins ec2 서버에서 ssh key 생성
```
ssh-keygen
```
2. 위에서 생성한 key와 github repository를 연결한다. setting -> deploy keys에 들어가서 key 등록을 해준다.
    - .pub이 붙은 파일을 넣어준다.(ssh-rsa로 시작)
3. jenkins를 들어가서 jenkins 관리 -> manage credentials 새로운 credential을 등록한다.
    - global credentials 등록
        - kind : ssh username with private key 선택
        - scope : global
        - ID : 입력하지 않아도 된다.
        - Username : Credential의 식별자
        - Private Key : Enter directly를 선택 후 위에서 생성한 키를 넣어준다. 
4. jenkins 아이템에서 구성에 들어가서 git repository 주소를 바꿔준다.
    - 기존 주소는 https git 주소이다. 이를 SSH git 주소로 바꿔준다.
    - 바꿔준뒤 아래 credentials을 위에서 생성한 credential로 바꿔준다.


### git private repository로 바꾸면서 제한이 생김
- git action 사용 불가
- PR reviewer 기능 사용 불가
- PR branch rule 사용 불가
- 이슈 담당자 여러명 등록 불가
- 등 여러가지 제한이 많이 생겼다. 이를 해결하기 위해선 organization 사람 수 마다 4$를 내는 Team plan 비용을 지불해야한다.
