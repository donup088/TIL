## Git Base
---
### 설정 & 스테이징
- Git config
```
git config --global user.name "USERNAME" 
git config --global user.email "EMAIL"

```
- 생성한 파일 스테이지에 추가하기
```
git add 파일이름
```
- 스테이징 취소하기 (언스테이징)
```
git reset 파일명
```
- git log 간결한 결과 보기
```
git log --oneline --graph --all --decorate
```

 ### 변경사항 적용하기
 - 원격저장소의 변경사항을 워킹트리에 반영(git fetch + git merge)
 ```
 git pull
 ```
 ### Git push 를 잘못했을 때
 - 소스 트리 활용 방법
    ```
    1. 잘못 push 한 곳에 이 커밋까지 현재 브랜치 초기화(Hard)
    2. 그 전 커밋을 클릭하고 이 커밋까지 현재 브랜치 초기화(Hard)
    3. 강제 push
    ```
### Git ignore
- ignore 적용이 안될 때
    ```
    git rm -r --cached .
    git add .
    git commit -m "커밋 메세지"
    ```
