### 스택
- 파이썬에서는 append()와 pop()을 사용하여 스택을 구현한다.
- append() : 리스트의 가장 뒤쪽에 데이터 삽입
- pop() : 리스트의 가장 뒤쪽에서 데이터를 꺼낸다.

### 큐
- 파이썬에서는 deque()를 사용하여 Queue를 구현한다.
    - deque()를 사용하기 위해 collections 모듈을 import한다.
    ```
    from collections import deque
    ```
- append() : 리스트의 가장 뒤쪽에 데이터 삽입
- popleft() : 가장 왼쪽의 데이터 추출
### 인접 행렬
```
graph=[
    [0,7,5],
    [7,0,INF],
    [5,INF,0]
    ]
```
### 인접 리스트
- 2차원 리스트를 사용한다.
```
graph=[[] for _ in range(3)]
# 노드 0에 연결된 노드 정보 저장(노드,거리)
graph[0].append((1,7))
graph[0].append((2,5))

graph[1].append((0,7))

graph[2].append((0,5))

print(graph)
```

### DFS 
- 동작 과정(Stack or 재귀 함수 사용)
  1. 탐색 시작 노드를 스택에 삽입하고 방문 처리
  2. 스택의 최상단 노드에 방문하지 않은 인접한 노드가 하나라도 있다면 그 노드를 스택에 넣고 방문 처리하고 방문하지 않은 인접 노드가 없다면 스택에서 최상단 노드를 꺼낸다.
  3. 2번의 과정을 수행할 수 없을 때까지 반복
```
def dfs(graph,v,visited):
  visited[v]=True
  print(v, end=' ')
  for i in graph[v]:
    if not visited[i]:
      dfs(graph,i,visited)
```

### BFS
- 동작 과정(Queue 사용)
1. 탐색 시작 노드를 큐에 삽입하고 방문 처리
2. 큐에서 노드를 꺼낸 뒤 해당 노드의 인접 노드 중에서 방문하지 않은 노드를 모두 큐에 삽입하고 방문 처리
3. 2번의 과정을 더 이상 수행할 수 없을 때까지 반복
- DFS보다 수행시간이 좋다.
```
def bfs(graph,start,visited):
  queue=deque([start])
  visited[start]=True
  while queue:
    v=queue.popleft()
    print(v, end=' ')
    for i in graph[v]:
      if not visited[i]:
        queue.append(i)
        visited[i]=True
```