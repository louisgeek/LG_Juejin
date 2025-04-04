# Java 排序算法之选择排序
- 选择排序是一种简单直观的选择排序算法，
每一趟从待排序的数据元素中选出最小（或最大）的一个元素，顺序放在已排好序的数列的最后，直到全部待排序的数据元素排完
通过对数组中的元素进行多次选择比较，每次从未排序的元素中选择最小（或最大）的元素放到已排序的末尾
每一次从待排序的数据元素中选出最小（或最大，取决于排序顺序）的一个元素，存放在序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到全部待排序数据元素的个数为零
过多次遍历未排序部分，每次找到最小（或最大）元素并放到已排序部分的末尾，直到所有元素有序
首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕
通过逐步寻找未排序序列中的最小（或最大）元素，并将其放置在已排序序列末尾的简单排序算法。其核心流程分为两个阶段：首先遍历未排序部分找到最小元素索引，然后将其与未排序部分的起始位置元素交换。每轮操作后，已排序序列长度增加一位，未排序序列减少一位，直到所有元素有序



重复地遍历要排序的数组，一次比较相邻的两个元素，通过比较两个元素并交换位置，如果顺序错误（不符合预期顺序）就把他们交换过来，将较大（或较小）的元素逐渐 “冒泡” 到数组末尾（让较大的数往下沉，较小的往上冒），直到没有再需要交换的元素为止
- 冒泡排序属于比较类排序
- 冒泡排序易于理解和实现，但效率较低
- 时间复杂度：平均 O(n^2) 最好 O(n) 最坏 O(n^2)
- 空间复杂度：O(1)
- 稳定性：稳定

```java
public static void main(String[] args) {
    int[] arr = {3, 5, 1, 7, 6, 2, 4};
    //外层循环遍历的次数控制排序的轮数，表示需要进行多少轮比较，每经过一轮排序最大的元素会逐渐 “冒泡” 到数组的末尾
    for (int i = 0; i < arr.length - 1; i++) {
        //内层循环进行相邻元素的比较和交换，每次循环结束后都会把未排序部分的最大元素移动到它的最终位置，后续的比较就不需要再包括这个元素，所以内层循环的比较范围会逐渐减小
        for (int j = 0; j < arr.length - 1 - i; j++) {
            //利用 - i 进行递减
            if (arr[j] > arr[j + 1]) { //升序
                //如果前面的元素比后面的元素大，则交换
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
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