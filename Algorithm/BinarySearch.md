### 이진 탐색 알고리즘
- 정렬되어 있는 리스트에서 탐색 범위를 절반씩 좁혀가며 데이터를 탐색하는 방법
- 시간 복잡도: O(logN)
- 큰 탐색 범위를 보면 이진 탐색을 떠올리자. ex) 0~10억
```
public static int binarySearch(int[] arr, int target, int start, int end) {
    while (start <= end) {
        int mid = (start + end) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] > target) end = mid - 1;
        else start = mid + 1;
    }
    return -1;
}
```