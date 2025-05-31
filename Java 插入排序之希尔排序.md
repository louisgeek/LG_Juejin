# Java 插入排序之希尔排序
- 希尔排序是直接插入排序的优化改进版（更加高效，也称为缩小增量排序），把数组按 gap 间隔（增量、步长）分成多个子序列，对每个子序列分别进行直接插入排序
- 初始间隔通常为数组长度的一半，然后逐步缩小间隔，随着数组逐步趋于有序，数据移动的次数逐步减少，直到最后间隔为 1，最终实现高效排序
- 希尔排序在性能上显著优于直接插入排序，适合中等规模数据，尤其是部分有序的数据
- 时间复杂度：平均 优于 O(n^2) 最好 O(log n) 最坏 O(n^2)
- 空间复杂度：O(1)
- 稳定性：不稳定

```java
public static void main(String[] args) {
    int[] arr = {3, 5, 1, 7, 6, 2, 4};
    //初始间隔为数组长度的一半
    int gap = arr.length / 2;
    while (gap >= 1) {
        //对每个间隔为 gap 的子序列进行直接插入排序
        for (int i = gap; i < arr.length; i++) {
            int key = arr[i]; //当前待插入元素 key
            int j = i - gap;
            //从后向前比较，找到插入位置并移动元素（将已排序部分中比 key 大的元素后移 gap 个位置）
            while (j >= 0 && arr[j] > key) {
                arr[j + gap] = arr[j];
                j -= gap;
            }
            //插入元素 key
            arr[j + gap] = key;
        }
        //逐步缩小间隔
        gap = gap / 2;
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