### 냅색 알고리즘 중복 허용
```
if __name__=="__main__":
  n,m=map(int,input().split())
  dy=[0]*(m+1)
  for i in range(n):
    w, v=map(int,input().split())
    for j in range(w,m+1):
      dy[j]=max(dy[j],dy[j-w]+v)
print(dy[m])
```

### 냅색 알고리즘 중복 불가능
```
if __name__=="__main__":
  n,m=map(int,input().split())
  dy=[0]*(m+1)
  for i in range(n):
    ps,pt=map(int,input().split())
    for j in range(m,pt-1,-1):
      dy[j]=max(dy[j],dy[j-pt]+ps)
print(dy[m])
```