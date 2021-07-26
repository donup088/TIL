## Vue, Spring boot 배포 자동화
- 사용 기술
1. Vue, Spring boot
2. Aws s3
3. Aws codeDeploy
4. Aws RDS
5. Travis CI
6. Nginx

### Spring boot 의 외부 security 파일 등록하기
- Spring boot에서 사용할 운영 DB 설정이나 보안과 관련된 부분은 서버에서 저장하고 저장한 것을 application.yml에서 사용할 수 있도록 한다.
- jar 파일을 실행할 때 스프링 설정 파일 위치를 지정한다.
    - classpath가 붙으면 resources 디렉토리 기준으로 경로가 생성된다.
```
nohup java -jar \ 
    -Dspring.config.location=classpath:/application.yml, /home/ec2-user/app/application-real-db.yml \
    $REPOSITORY/$JAR_NAME 2>&1 &
```

### frontend, backend 하나의 respository에서 배포 자동화 시키기
- AWS IAM 생성
    - Travis CI가 AWS의 S3 , CodeDeploy에 접근할 수 있도록 한다.
    - AmazonS3FullAccess, AWSCodeDeployFullAccess 권한 부여
    - AWS access key, secrey key를 받아서 travis 환경변수로 설정한다.
- AWS s3 버킷 생성
- EC2가  CodeDeploy를 연동 받을 수 있게 IAM 역할 추가
    - 역할 만들기 , AmazonEC@RoleforAWSCodeDeploy 추가
    - ec2에 IAM 수정
    - ec2에 접속해서 CodeDeploy 에이전트 설치
    ```
    aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2

    chmod +x ./install

    sudo ./install auto
    ```
- CodeDeploy에서 Ec2에 접근하기 위한 권한 생성
    - 역할 만들기, AWSCodeDeployRole 생성
- CodeDeploy 생성
    - 애플리케이션 생성 , 배포 그룹 생성
- Travis CI 연동
    - .travis.yml 
    ```
    matrix:
        include:
            - language: java
            jdk:
                - openjdk11
            branches:
                only:
                - master

            cache:
                directories:
                - '$HOME/.m2/repository'
                - '$HOME/.gradle'

            before_install:
                - cd backend/
                - chmod +x gradlew

            script: "./gradlew clean build"

            before_deploy:
                - mkdir -p before-deploy
                - cp scripts/*.sh before-deploy/
                - cp appspec.yml before-deploy/
                - cp build/libs/*.jar before-deploy/
                - cd before-deploy && zip -r before-deploy *
                - cd ../ && mkdir -p deploy
                - mv before-deploy/before-deploy.zip deploy/Taskagile-Vue-Springboot.zip

            deploy:
                - provider: s3
                access_key_id: $AWS_ACCESS_KEY
                secret_access_key: $AWS_SECRET_KEY
                bucket: s3 버킷 이름
                region: ap-northeast-2
                skip_cleanup: true
                acl: private
                local_dir: deploy
                wait-until-deployed: true

                - provider: codedeploy
                access_key_id: $AWS_ACCESS_KEY
                secret_access_key: $AWS_SECRET_KEY
                bucket: taskagile-backend-build
                key: Taskagile-Vue-Springboot.zip # 빌드 파일 압축해서 전달

                bundle_type: zip # 압축 확장자
                application: 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
                deployment_group: 웹 콘솔에서 등록한 CodeDeploy 배포그룹
                region: ap-northeast-2
                wait-until-deployed: true

            - language: node.js
            node_js:
                - "stable"

            before_install:
                - cd frontend/

            script:
                - npm run build

            before_deploy:
                - mkdir -p before-frontend-deploy
                - cp scripts/*.sh before-frontend-deploy/
                - cp appspec.yml before-frontend-deploy/
                - cp -r dist before-frontend-deploy
                - cd before-frontend-deploy && zip -r before-frontend-deploy *
                - cd ../ && mkdir -p deploy-frontend
                - mv before-frontend-deploy/before-frontend-deploy.zip deploy-frontend/front-end.zip

            deploy:
                - provider: s3
                access_key_id: $AWS_ACCESS_KEY
                secret_access_key: $AWS_SECRET_KEY
                bucket: s3 버킷 이름
                region: ap-northeast-2
                skip_cleanup: true
                acl: private
                local_dir: deploy-frontend
                wait-until-deployed: true

                - provider: codedeploy
                access_key_id: $AWS_ACCESS_KEY
                secret_access_key: $AWS_SECRET_KEY
                bucket: s3 버킷 이름
                key: front-end.zip
                bundle_type: zip
                application: 웹 콘솔에서 등록한 CodeDeploy 애플리케이션
                deployment_group: 웹 콘솔에서 등록한 CodeDeploy 배포그룹
                region: ap-northeast-2
                wait-until-deployed: true

    notifications:
    email:
        recipients:
        - 이메일
    ```
- appspec.yml 파일 생성
    - backend
        ```
        version: 0.0
        os: linux
        files:
          - source: /
            destination: /home/ec2-user/app/step2/zip/ #빌드 파일이 들어갈 경로
            overwrite: yes

        permissions:
          - object: /
            pattern: "**"
            owner: ec2-user
            group: ec2-user

        hooks:
        ApplicationStart:
          - location: deploy.sh
            timeout: 120
            runas: ec2-user
        ```
    - frontend
        ```
        version: 0.0
        os: linux
        files:
          - source: /
            destination: /home/ec2-user/app/step2/front-end-zip/
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
- 배포 스크립트 작성 
    - backend
        ```
        REPOSITORY=/home/ec2-user/app/step2
        PROJECT_NAME=TaskAgile-Vue-Springboot

        echo "> Build 파일 복사"

        cp $REPOSITORY/zip/*.jar $REPOSITORY/

        echo "> 현재 구동중인 애플리케이션 pid 확인"

        CURRENT_PID=$(pgrep -fl backend | grep java  | awk '{print $1}')

        echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"

        if [ -z "$CURRENT_PID" ]; then
            echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
        else
            echo "> kill -15 $CURRENT_PID"
            kill -15 $CURRENT_PID
            sleep 5
        fi

        echo "> 새 어플리케이션 배포"

        JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

        echo "> JAR Name: $JAR_NAME"

        echo "> $JAR_NAME 에 실행권한 추가"

        chmod +x $JAR_NAME

        echo "> $JAR_NAME 실행"
        nohup java -jar \
            -Dspring.config.location=classpath:/application.yml,/home/ec2-user/app/application-real-db.yml \
            -Dspring.profiles.active=prod \
            $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
        ```
    - frontend
        ```
        #!/bin/bash

        REPOSITORY=/home/ec2-user/app/step2

        echo "> zip 파일 복사 "

        sudo cp -r $REPOSITORY/front-end-zip/dist /var/www/html

        echo "> nginx restart"

        sudo service nginx restart
        ```