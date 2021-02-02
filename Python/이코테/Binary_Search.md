### 이진 탐색
```
def binary_search(array,target,start,end):
  if start>end:
    return None
  mid=(start+end)//2
  if array[mid] == target:
    return mid
  elif array[mid]>target:
    return binary_search(array,target,start,mid-1)
  else:
    return binary_search(array,target,mid+1,end)
    

n, target=list(map(int,input().split()))
array=list(map(int,input().split()))

result=binary_search(array,target,0,n-1)
if result==None:
  print("원소가 존재하지 않습니다.")
else:
  print(result+1)
```

### 빠르게 입력받기
- 이진탐색을 사용한는 문제에서 입력데이터양이 많을 경우 사용한다.
- sys 라이브러리를 사용한다
```
import sys

input_data=sys.stdin.readline().rstrip()
print(input_data)
```