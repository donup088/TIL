### DFS(깊이 우선 탐색)
---
- 동작 과정(Stack or 재귀 함수 사용)
  1. 탐색 시작 노드를 스택에 삽입하고 방문 처리
  2. 스택의 최상단 노드에 방문하지 않은 인접한 노드가 하나라도 있다면 그 노드를 스택에 넣고 방문 처리하고 방문하지 않은 인접 노드가 없다면 스택에서 최상단 노드를 꺼낸다.
  3. 2번의 과정을 수행할 수 없을 때까지 반복

- java 예시
```
 public static void dfs(int x){
        visited[x]=true;
        for(int i=0;i<graph.get(x).size();i++){
            int y=graph.get(x).get(i);
            if(!visited[y]) dfs(y);
        }
    }
```

- Tip
    1. java에서 이차원 리스트를 사용
    ```
    public static ArrayList<ArrayList<Integer>> graph = new ArrayList<ArrayList<Integer>>();
    ```

 
