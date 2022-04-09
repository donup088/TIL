spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: 
    username: 
    password: 
    hikari:
      maximum-pool-size: 15
      idle-timeout: 10000
      connection-timeout: 10000
      max-lifetime: 580000
      validation-timeout: 10000
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: 
            client-secret: 
            scope: profile, email
  mail:
    host: smtp.gmail.com
    port: 587
    username: 
    password: 
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true

kakao:
  client:
    id: 
    secret: 

jwt:
  access-token-props:
    secret: 
    expiration-time-milli-sec: 3600000
  refresh-token-props:
    secret: 
    expiration-time-milli-sec: 86400000

decorator:
  datasource:
    p6spy:
      enable-logging: false
      multiline: false

oauth2:
  success:
    redirect:
      url:

dev:
  server:
    url: 

s3:
  file:
    prefix: /tmp/

sms:
  serviceId: 
  accessKey: 
  secretKey: 
  from: 

crypto:
  key: 


cloud:
  aws:
    credentials:
      accessKey: 
      secretKey: 
    s3:
      bucket: 
      public: 
    region:
      static: ap-northeast-2
    stack:
      auto: false