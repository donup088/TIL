## spring boot 배포시 생겼던 이슈들

### elb, jenkins, spring boot 배포시 logback 설정이 null로 나오고 파일 생성이 안되는 이슈
- 처음에는 무슨 문제인지 몰라서 한참 알아봄...
- logback.spring.xml 파일에서 log 생성 위치 설정에서 권한문제 발생(권한문제에 대한 오류메세지가 따로 안나온다)
- log 생성 위치를 /home/ec2-user/app/dev/logs 로 했더니 문제 해결 
    - 여기서 logs 파일은 미리 ec2-user 권한으로 만들어두었다. 상대경로로 했더니 파일 생성이 안되서 절대경로로 바꿨다.
- 배포시 logback 설정이 null로 되는 이유는 spring-logback.xml 파일이 없거나 파일이 생성되지 않을 때 나타나는 것으로 보인다.

### elb,codedeploy 사용시 대상그룹이 없어지는 이슈
- 대상그룹이 없어지는 이유를 몰라서 계속 추가해주며 왜그런지 알아보았다.
- codedeploy에서 elb에 배포할 때 배포가 모두 성공하면 대상그룹이 다시 생겼다..
- 삽질중 알아냈던 정보들
    - elb healthcheck 옵션
        - Healthy Threashold : Unhealthy로 판정난 서버가 몇번 연속으로 정상으로 나오면 정상적인 인스턴스로 볼 것인지 설정
        - UnHealthy Threashold : 기본값이 2이므로 2번 연속 unHealthy의 조건을 충족하면 unHealthy로 판정
        - Timeout : 몇 초내에 응답하지 않으면 비정상으로 판단
        - Interval : Health Check를 하는 시간 , 기본값 30초
    - codedeploy 로 elb 배포시 속도 올리기
        - BlockTraffic 을 줄이기 위해 Deregistration delay 값을 낮추면 된다. 기본값은 300초이다.
        - allowTraffic 구간을 줄이기 위해 Healthy threshold 를 5 -> 2 로 변경하고 interval도 30 -> 10 으로 설정한다. (대상 그룹의 health check 할 때 오래 걸리기 때문)
- elb를 사용하여 frontend, backend 모두 무중단배포를 하려고 했지만 인스턴스를 더 만들어야 가능하다는 사실을 알게되었다. 개발서버를 구축하는 상황이라 인스턴스를 최소한으로 사용해도 될 것 같다는 생각이 들어서 elb를 없애고 https 를 let's encrypt로 사용하고 codedeploy에서 로드밸런싱을 없애주었다.

### spring boot s3 업로드 jenkins 배포시 IOException 발생
- 처음에 해당 에러를 customexception으로 처리하고 있었기 때문에 무엇이 원인인지 알아낼 방법이 없었다.
- CustomException을 만들어 사용하더라도 에러 로그는 사용해야 원인을 알 수 있다.
- 로그를 통해 확인해보니 파일 생성에 있어서 permission 이 원인이었다.
- jenkins로 배포하지 않고 수동배포할 때는 ec2에 파일 생성권한이 있어서 에러가 안났지만 jenkins로 배포하니 배포하는 유저가 jenkins 이기 때문에 ec2에 파일 생성권한이 없었다.
- 기존에는 파일 생성 경로를 파일 이름으로 했었지만 앞에 "/tmp/"를 추가해주어서 파일을 생성할 수 있도록하여 해결하였다.
