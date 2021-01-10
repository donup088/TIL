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

    ```
    git log --oneline --all
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
### revert 사용
- 커밋을 되돌리고 싶을 때
- 최신 커밋부터 취소하는 것이 좋다.
    ```
    git revert 해당커밋
    ```
- 해당하는 커밋을 취소하는 커밋을 만들어낸다.
---
### tag 사용
- 주석이 있는 태그 작성
    ```
    git tag -a -m "태그 생성 주석부분" v0.1
    git log --oneline 
    git push origin v0.1
    ```
### Branch
- 브랜치 생성
    ```
    git branch 브랜치이름
    ```
- 브랜치 삭제(-D로 설정하면 강제로 삭제)
    ```
    git branch -d 브랜치이름
    ```
- git reset --hard 브랜치 되돌리기
    ```
    git reset --hard HEAD~2 //master를 두 커밋 이전으로 옮김
    ```
    -  git reset --hard 명령어는 아래 세 명령을 한 번에 수행하는 것과 같다.
    ```
    git checkout HEAD~2
    git branch -f master
    git checkout master
    ```
- git rebase 사용
    - 현재 브랜치에만 있는 새로운 커밋을 대상 브랜치 위로 재배치 시킨다.
    - 재배치할 커밋이 없는 경우 아무런 동작을 하지 않는다.
    - 빨리 감기 병합이 가능한 경우 rebase 명령은 빨리 감기 병합을 한다.
    ```
    git rebase <대상 브랜치>
    ```
    - rebase로 가지 없애기
    ```
    git reset --hard HEAD~
    git rebase origin/master
    ```
    - 주의할점
        - 원격 저장소에 push한 브랜치는 rebase 하지 않는다. (로컬 브랜치에서만 사용한다.)
    
    



