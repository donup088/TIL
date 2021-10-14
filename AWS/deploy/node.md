## node 사용 배포시 이슈

### react 빌드시 힙메모리 부족으로 에러 발생
- .env 파일에 설정 추가
    ```
    NODE_OPTIONS="--max-old-space-size=4096"
    GENERATE_SOURCEMAP=false
    ```
    - 빌드시 메모리를 더 할당하도록 설정해준다.