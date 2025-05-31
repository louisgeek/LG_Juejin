# Java 插入排序之二分法插入排序
- 二分法插入排序是直接插入排序的优化改进版（相对高效，也称为折半插入排序），在直接插入排序的基础上，用二分法查找替代线性顺序查找来确定插入位置​，从而减少比较次数
- 二分法插入排序在性能上优于直接插入排序，适合小规模或部分有序的数据
- 时间复杂度：平均 O(n^2) 最好 O(log n) 最坏 O(n^2)
- 空间复杂度：O(1)
- 稳定性：稳定

```java
/**
待排序： 3 5 1 7 6 2 4
第 1 轮：3 5 1 7 6 2 4 剩余 1 7 6 2 4
第 2 轮：1 3 5 7 6 2 4 剩余 7 6 2 4
第 3 轮：1 3 5 7 6 2 4 剩余 6 2 4
第 4 轮：1 3 5 6 7 2 4 剩余 2 4
第 5 轮：1 2 3 5 6 7 4 剩余 4
第 6 轮：1 2 3 4 5 6 7
...
*/
public static void main(String[] args) {
    int[] arr = {3, 5, 1, 7, 6, 2, 4};
    //外层循环从第 2 个元素开始遍历
    for (int i = 1; i < arr.length; i++) {
        int key = arr[i]; //当前待插入元素 key
        int left = 0;
        int right = i - 1;
        //使用二分法查找确定插入位置
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (arr[mid] < key) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        //找到插入位置并移动元素（将 i-1 到 left 的元素后移）
        for (int j = i - 1; j >= left; j--) {
            arr[j + 1] = arr[j];
        }
        //插入元素 key
        arr[left] = key;
    }
    //
    System.out.println("排序后的数组：");
    for (int num : arr) {
        System.out.print(num + " ");
    }
}
//---------------------------------
排序后的数组：
1 2 3 4 5 6 7 
```