### 내장 함수
- sum([1,2,3,4]) 1~4를 더한다.
- min(1,2,3) 1,2,3 중 가장 작은 수를 찾는다.
- max(1,2,3)
- eval("(3+5)*7") 문자열 형태로 주어진 수식을 계산한다.
- result=sorted([3,7,2,3,1]) 오름차순 정렬
- result=sorted([3,7,2,3,1], reverse=True) 내림차순 정렬

### itertools
- permutations : 순열 계산
- 리스트에서 3개를 뽑아 나열하는 모든 경우의 수
    ```
    from itertools import permutations

    data=['A','B','C']
    result=list(permutations(data,3))
    print(result)
    ```
- 2개를 뽑아 순서에 상관없이 나열하는 모든 경우의 수
    ```
    from itertools import combinations

    data=['A','B','C']
    result=list(combinations(data,2))
    print(result)
    ```

### heapq
- 힙에 원소를 삽입할 때는 heapq.heappush() 사용
- 원소를 꺼낼 때는 heapq.heappop() 사용

### bisect
- bisect_left(a,x): 정렬된 순서를 유지하면서 리스트 a에서 데이터 x를 삽입할 가장 왼쪽 인덱스를 찾음
- bisect_right(a,x): 정렬된 순서를 유지하면서 리스트 a에서 데이터 x를 삽입할 가장 오른쪽 인덱스를 찾음

### Collections
- 파이썬에서는 큐를 deque를 사용해 구현한다.
    - popleft() : 첫 번째 원소 제거
    - pop() : 마지막 원소 제거
    - appendleft(x) : 첫 번째 인덱스에 x 삽입
    - append(x) : 마지막 인덱스에 x 삽입
    - 큐 구현 - append() ,popleft() 사용
- Counter : 해당 객체 내부의 원소가 몇 번 등장했는 지 알 수 있다.

### math 
- math.factorial(5) 5!사용가능
- math.sqrt(7) 7의 제곱근
- math.gcd(21,14) 21,14의 최대 공약수
- math.pi , math.e 사용가능
