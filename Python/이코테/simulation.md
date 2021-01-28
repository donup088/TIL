### 구현
- 완전탐색 : 모든 경우의 수를 다 계산하는 해결방법
- 시뮬레이션 : 문제에서 제시한 알고리즘을 한 단계씩 차례대로 직접 수행

- if 문에서 in을 사용할 수 있다.
```
# 반복문 3개를 돌면서 str(i),str(j),str(k)에 3이 있는 지 판단할 수 있다.
...
if '3' in str(i) + str(j) + str(k):
    count += 1
```

- 문제에서 a1로 행렬이 표현될 경우 a1을 숫자로 바꾸기 위한 방법
```
input_data=input()
row=int(input_data[1])
column=int(ord(input_data[0])-int(ord('a')))+1
```

### 방향을 설정하는 문제
- N X M 행렬 초기화
    ```
    n, m =map(int,input().split())
    d=[[0]*m for _ in range(n)]
    ```
- 행렬 입력 받기
    ```
    arr=[]
    for i in range(n):
        arr.append(list(map(int,input().split())))
    ```
- dx,dy라는 별도의 리스트를 만들어 방향을 정하는 것이 좋다.