### 선택 정렬
- 처리되지 않은 데이터중에 가장 작은 데이터를 선택해 맨 앞에 있는 데이터와 바꾸는 것을 반복한다.
```
public class SortExample {
    public static void main(String[] args) {
        int n = 10;
        int[] arr = {7, 5, 9, 0, 3, 1, 6, 2, 4, 8};
        for (int i = 0; i < n; i++) {
            int min_index = i;
            for (int j = i + 1; j < n; j++) {
                if (arr[min_index] > arr[j]) {
                    min_index = j;
                }
            }
            int tmp = arr[i];
            arr[i] = arr[min_index];
            arr[min_index] = tmp;
        }
        for (int i = 0; i < n; i++) {
            System.out.println(arr[i] + " ");
        }
    }
}
```
- 시간 복잡도: O(N^2)
### 삽입 정렬
- 처리 되지 않은 데이터를 하나씩 골라 적절한 위치에 삽입한다.
```
public class InsertSortExample {
    public static void main(String[] args) {
        int n = 10;
        int[] arr = {7, 5, 9, 0, 3, 1, 6, 2, 4, 8};
        for (int i = 1; i < n; i++) {
            //한 칸씩 왼쪽으로 이동
            for (int j = i; j > 0; j--) {
                if (arr[j] < arr[j - 1]) {

                    int tmp = arr[j];
                    arr[j] = arr[j - 1];
                    arr[j - 1] = tmp;
                } else break;
            }
        }
        for (int i = 0; i < n; i++) {
            System.out.println(arr[i] + " ");
        }
    }
}

```
- 시간 복잡도: O(N^2)

### 퀵 정렬
- 기준 데이터를 설정하고 그 기준보다 큰 데이터와 작은 데이터의 위치를 바꾸는 방법
- 평균의 경우 시간 복잡도: O(NlogN)
- 최악의 경우 시간 복잡도: O(N^2)
    - ex) 이미 정렬된 배열
```
public class QuickSortExample {
    public static void quickSort(int[] arr, int start, int end) {
        if (start >= end) return;
        int pivot = start;
        int left = start + 1;
        int right = end;
        while (left <= right) {
            while (left <= end && arr[left] <= arr[pivot]) left++;
            while (right > start && arr[right] >= arr[pivot]) right--;
            // 엇갈렸다면 작은 데이터와 피벗을 교체
            if (left > right) {
                int tmp = arr[pivot];
                arr[pivot] = arr[right];
                arr[right] = tmp;
            } else {
                int tmp = arr[left];
                arr[left] = arr[right];
                arr[right] = tmp;
            }
        }
        quickSort(arr, start, right - 1);
        quickSort(arr, right + 1, end);
    }

    public static void main(String[] args) {
        int n = 10;
        int[] arr = {7, 5, 9, 0, 3, 1, 6, 2, 4, 8};
        quickSort(arr, 0, n - 1);
        for (int i = 0; i < n; i++) {
            System.out.println(arr[i] + " ");
        }
    }
}
```

### 계수 정렬
- 테이터의 크기 범위가 제한되어 정수 형태로 표현할 수 있을 때 사용가능
- 동일한 값을 가지는 데이터가 많을 때 유용하다.
```
public class CountSortExample {
    public static final int MAX_VALUE = 9;

    public static void main(String[] args) {
        int n = 15;
        int[] arr = {7, 5, 9, 0, 3, 1, 6, 2, 4, 8, 4, 7, 5, 3, 1};
        int[] cnt = new int[MAX_VALUE + 1];
        for (int i = 0; i < n; i++) {
            cnt[arr[i]] += 1;
        }
        for (int i = 0; i <= MAX_VALUE; i++) {
            for (int j = 0; j < cnt[i]; j++) {
                System.out.println(i + " ");
            }
        }
    }
}
```

### java 정렬 함수
```
    Arrays.sort(a); //a배열을 오름차순 정렬
    Arrays.sort(a, Collections.reverseOrder()); //a 배열을 내림차순 정렬
```
- Arrays.sort(a, Collections.reverseOrder()) 사용할 때 배열 a는 Wrapper 클래스로 해야한다. ex)Integer  

