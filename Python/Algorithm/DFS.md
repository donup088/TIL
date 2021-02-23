### 부분집합 만들기
- 부분집합 만드는 부분이 사용될 때 DFS를 사용하면 좋다.


### DFS 사용
- 입력되는 값들을 리스트로 만들기
    ```
    c,n=map(int,input().split())
    a=[0]*n
    for i in range(n):
        a[i]=int(input())
    ```
- DFS에 시작되는 인덱스값과 sum을 주고 문제 해결
    - 시간초과가 날 경우 가지를 더 짜르기 위한 방법을 찾아야한다.
    - 가지를 더 자르기 위한 tsum 사용
    ```
    def DFS(L,sum,tsum):
    global result
    if sum+(total-tsum)<result:
        return
    if sum>c:
        return 
    if L==n:
        if sum>result:
        result=sum
    else:
        DFS(L+1,sum+a[L],tsum+a[L])
        DFS(L+1,sum,tsum+a[L])
    ```

### 중복순열을 활용하기
```
def DFS(L,sum):
  global res
  if L>res:
    return
  if sum>m:
    return
  if sum==m:
    if L<res:
      res=L
  else:
    for i in range(n):
      DFS(L+1,sum+a[i])
  
if __name__=="__main__":
  n=int(input())
  a=list(map(int,input().split()))
  m=int(input())
  res=2147000000
  a.sort(reverse=True)
  DFS(0,0)
```
- 중복순열의 경우 DFS 함수에서 반복문을 사용하여 중복가능한 만큼 재귀호출을 한다.

### 순열 만들기
- 중복순열을 구현하던 방법에서 체크리스트만 만들어서 중복 체크만 해주면 된다.
```
global cnt
  if L==m:
    for j in range(L):
      print(res[j],end=' ')
    print()
    cnt+=1
  else:
    for i in range(1,n+1):
      if ch[i]==0:
        ch[i]=1
        res[L]=i
        DFS(L+1)
        ch[i]=0
if __name__=="__main__":
  n,m=map(int,input().split())
  res=[0]*n
  ch=[0]*(n+1)
  cnt=0
  DFS(0)
```