# AWS 기본설정

## AWS 설정하기
- 포트 항목이 22인 경우: AWS를 EC2 터미널로 접속하는 것을 의미
  - 보안 그룹에서 ssh접속은 지정된 IP에서만 ssh접속이 가능하도록 구성하는 것이 안전하다. 
- 고정 IP 등록: 고정 IP 할당을 하지 않으면 인스턴스의 IP가 인스턴스를 중지하고 다시 시작하면 변경된다.
  - EIP 할당(탄력적 IP 할당): 탄력적 IP를 발급 받고 EC2 주소를 연결한다.(작업 -> 주소 연결 선택) 
  - 탄력적 IP를 생성하고 EC2에 바로 연결하지 않으면 비용 청구가 된다. 탄력적 IP를 사용할 인스턴스가 없다면 탄력적 IP를 삭제해야한다.
  
## putty 사용
- puttygen을 통해 pem키를 ppk파일로 변환해야한다.
- putty HostName: username@public_ip ex)ec2-user@탄력적 IP 주소
- Connection -> SSH -> Auth 를 들어가서 ppk 파일 등록
- Session에서 현재 설정들 저장

## 아마존 리눅스 서버 생성 시 꼭 해야 할 설정들
- java 설정
  ```
  sudo yum install -y java-1.8.0-openjdk-devel.x86_64 //java 8 설치
  ```
  ```
  sudo /usr/sbin/alternatives --config java //java 버전 선택
  ```
  ```
  sudo yum remove java-1.7.0-openjdk //사용하지 않는 java버전 삭제
  java -version //현재 버전 확인
  ```
- 타임존 변경
  ```
  sudo rm /etc/localtime
  sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
  date //타임존 확인
  ```
- Hostname 변경
  ```
  sudo hostnamectl set-hostname webserver.mydomain.com
  sudo reboot //재부팅후 hostname 변경 확인
  ```
  ```
  sudo vim /etc/hosts
  ```
  위의 명령어로 파일을 열고 아래 설정 추가
  ```
  127.0.0.1   등록한 HOSTNAME
  ```
  등록한 호스트이름 확인
  ```
  curl 등록한 호스트 이름
  ```
  