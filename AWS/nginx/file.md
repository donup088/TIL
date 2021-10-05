## Nginx 사용시 마주쳤던 문제들
- springboot로 파일업로드 용량을 제한해주었지만 실제로 적용이 안되었다. nginx 설정 추가로 해결
    ```
    http {
    # Set client upload size - 100Mbyte
    client_max_body_size 100M;
    ...
    ..
    .
    }
    ```
- nginx 설정을 바꾸고 재시작하면 적용된다.