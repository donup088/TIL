### 다익스트라 최단 경로 알고리즘
- 특정한 노드에서 출발하여 다른 모든 노드로 가는 최단 경로를 계산한다.
- 음의 간선이 없어야한다.
- 그리디 알고리즘으로 분류된다.
    - 매 상황에서 가장 비용이 적은 노드를 선택해 반복한다.
- 동작과정
1. 출발 노드를 설정한다.
2. 최단 거리 테이블을 초기화한다.
3. 방문하지 않은 노드 중에서 최단 거리가 가장 짧은 노드를 선택한다.
4. 해당 노드를 거쳐 다른 노드로 가는 비용을 계산하여 최단 거리 테이블을 갱신한다.
5. 3,4번을 반복한다.

### 복잡한 구현 개선
- 현재 가장 가까운 노드를 저장하기 위해 힙 자료구조를 이용한다.
- 최단 거리가 가장 짧은 노드를 선택하기 위해 최소 힙을 사용한다.
```
public class ShortestExample {
    static class Node implements Comparable<Node {
        private int index;
        private int distance;

        public Node(int index, int distance) {
            this.index = index;
            this.distance = distance;
        }

        public int getIndex() {
            return index;
        }

        public int getDistance() {
            return distance;
        }

        //거리가 짧은 것이 높은 우선순위를 가지도록 설정
        @Override
        public int compareTo(Node other) {
            if (this.distance < other.distance) {
                return -1;
            }
            return 1;
        }
    }

    public static final int INF = (int) 1e9;
    public static int n, m, start;
    public static ArrayList<ArrayList<Node>> graph = new ArrayList<ArrayList<Node>>();
    public static int[] d = new int[100001];

    public static void dijkstra(int start) {
        PriorityQueue<Node> pq = new PriorityQueue<>();
        d[start] = 0;
        while (!pq.isEmpty()) {
            Node node = pq.poll();
            int dist = node.getDistance(); //현재 노드까지의 비용
            int now = node.getIndex(); //현재 노드
            if (d[now] < dist) continue; // 현재 노드가 이미 처리된 적이 있는 노드라면 무시

            //현재 노드와 연결된 다른 인접한 노드들을 확인
            for (int i = 0; i < graph.get(now).size(); i++) {
                int cost = d[now] + graph.get(now).get(i).getDistance();
                //현재 노드를 거쳐서 다른 노드로 이도하는 거리가 더 짧은 경우
                if (cost < d[graph.get(now).get(i).getIndex()]) {
                    d[graph.get(now).get(i).getIndex()] = cost;
                    pq.offer(new Node(graph.get(now).get(i).getIndex(),cost));
                }
            }
        }
    }
    ...
```