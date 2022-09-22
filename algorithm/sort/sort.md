---
title: 排序算法整理
date: 2022-08-14
tags: [算法,排序]
toc: true
---

![image-20210710193034597](Sort/image-20210710193034597.png)

## 1、冒泡排序

- 冒泡排序：重复遍历数组，每次比较相邻的两个元素，如果顺序错误就进行交换，较小的元素会慢慢“冒泡”到最上边。

<!--more-->

- 时间复杂度
  - 最好：T(n) = O(n)	已排好序，无需交换。
  - 最坏：T(n) = O(n^2)
  - 平均：T(n) = O(n^2)

```java
	public static int[] bubbleSort(int[] arr) {
        int len = arr.length;
        for (int i = 0; i < len; i++) {
            // 外层循环每循环一次，就会有一个数被“冒泡”到数组顶端，确认位置，所以是 len-i-1。
            for (int j = 0; j < len - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
        return arr;
    }
```

## 2、选择排序

- 选择排序：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置；然后从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。
- 时间复杂度：表现比较稳定，对于任何数据的时间复杂度都是 O(n^2)。
  - 最好：T(n) = O(n^2)
  - 最坏：T(n) = O(n^2)
  - 平均：T(n) = O(n^2)

```java
    public static int[] selectionSort(int[] arr) {
        int len = arr.length;
        // 每次遍历都会在未排序序列中找出一个最小的数放到已排序序列后边，已排序长度加 1，未排序长度减 1。
        for (int i = 0; i < len; i++) {
            int minIndex = i;
            for (int j = i; j < len; j++) {
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            if (minIndex != i) {
                int temp = arr[i];
                arr[i] = arr[minIndex];
                arr[minIndex] = temp;
            }
        }
        return arr;
    }
```

## 3、插入排序

- 插入排序：插入排序的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序只需用到O(1)的额外空间，因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为新元素提供插入空间。

- 时间复杂度

  - 最好：T(n) = O(n)	已排好序，无需寻找插入位置。

  - 最坏：T(n) = O(n^2)

  - 平均：T(n) = O(n^2)

```java
	public static int[] insertionSort(int[] arr) {
        int len = arr.length;
        for (int i = 0; i < len - 1; i++) {
            int cur = arr[i + 1];
            int index = i;
            while (index >= 0 && arr[index] > cur) {
                arr[index + 1] = arr[index];
                index--;
            }
            arr[index + 1] = cur;
        }
        return arr;
    }
```

> 选择排序和插入排序的共同点是都将需要排序的数组分为已排序和未排序的两部分，操作都在这个数组内完成，不需要额外空间。
>
> 不同点是选择排序因为搜索到的结果就是未排序序列中最小的数，所以直接与已排序序列后第一个未排序的数交换即可；而插入排序因为是在未排序序列中依次寻找，没有搜索的过程，所以还要在已排序序列中搜索该数的位置。
>
> - 选择排序是在未排序的数据中直接找出最小（大）的元素，然后放到已排序序列中。
> - 插入排序是按顺序对未排序数据进行处理，然后在已排序的数据中寻找该数据的位置。

## 4、希尔排序

- 希尔排序：希尔排序是简单插入排序经过改进之后的一个更高效的版本，也是一种插入排序，也称为缩小增量排序。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫缩小增量排序。希尔排序是希尔于1959年提出的一种排序算法，同时该算法是冲破O(n2）的第一批算法之一。

  希尔排序是把记录按下表的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

- 时间复杂度

  最好：O(n logn)

  最坏：O((n logn)^2)

  平均：O((n logn)^2)

```java
 public static int[] shellSort(int[] arr) {
        int len = arr.length;
        int gap = len / 2;
        while (gap > 0) {
            for (int i = gap; i < len; i++) {  // 从 gap 开始遍历
                int temp = arr[i];  // 记录当前的值
                int index = i - gap;  // 找到当前下标的前 gap 个的值
                while (index >= 0 && arr[index] > temp) {  // 找到插入位置
                    arr[index + gap] = arr[index];
                    index -= gap;
                }
                arr[index + gap] = temp;  // 插入排序
            }
            gap /= 2;  // 缩小 gap
        }
        return arr;
    }
```

> **希尔排序**是插入排序的改进版本，弥补了插入排序在某些情况下的缺点。
>
> 例如，有一个长度为100的数组，前80个数是已经排好序的有序数组，第81个元素是1，在此时如果采用普通插入排序的话就会用第81个数去跟前面有序区域的所有元素比较大小，但恰巧第81个数又是这100个数里最小的，那么前面有序区域里的80个元素都要往后移动一个位置，这种情况就非常影响排序性能。
>
> 但是在某些极端情况下，希尔排序的性能没有直接插入排序性能好。比如对序列 `2 1 5 3 7 6 9 8` 进行排序，无论取增量为4还是2，数组都是已经排好序的，不用交换，必须要取增量为1才能继续排序。但增量为1的话就与普通的插入排序是一样的了。这样一来，希尔排序不但没有减少直接插入排序的工作量，反而白白增加了分组操作的成本。

每一轮希尔增量的增加的方式是等比的，这就使希尔增量存在盲区，所以有更加严谨合理的希尔增量增加方式，使用不同的增量对希尔排序的时间复杂度的改进也不一样。

```java
Hibbard增量：1，3，7，15 ...（2^k-1）	最坏：O(n^(3/2))
Sedgewick增量：1，5，19，41，109...（9*4^k-9*2^k+1 或 4^k-3*2^k+1）	最坏：O(n^(4/3))
```

## 5、归并排序

- 归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法的一个非常典型的应用。归并排序是一种稳定的排序方法。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为2-路归并。

- 时间复杂度

  最好：O(n)

  最坏：O(n logn)

  平均：O(n logn)

  > 和选择排序一样，归并排序的性能不受输入数据的影响，但表现比选择排序好的多，因为始终都是O(n log n）的时间复杂度。代价是需要额外的内存空间。

```java
    public static int[] mergeSort(int[] arr) {
        int len = arr.length;
        if (len == 1) {
            return arr;
        }
        int mid = len / 2;
        int[] left = Arrays.copyOfRange(arr, 0, mid);
        int[] right = Arrays.copyOfRange(arr, mid, len);
        return merge(mergeSort(left), mergeSort(right));
    }

    public static int[] merge(int[] left, int[] right) {
        int[] res = new int[left.length + right.length];
        int i = 0, j = 0;
        for (int index = 0; index < res.length; index++) {
            if (i >= left.length) {  // 左边指针走到尽头
                res[index] = right[j++];
            } else if (j >= right.length) {  // 右边指针走到尽头
                res[index] = left[i++];
            } else if (left[i] > right[j]) {  // 左边比右边大，按照从小到大的顺序排列，右边要放入到 res 中。
                res[index] = right[j++];  // 右边的指针指向下一位
            } else {
                res[index] = left[i++];  // 左边的指针指向下一位
            }
        }
        return res;
    }
```

## 6、快速排序

- 快速排序：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

- 时间复杂度

  最好：O(n logn)

  最坏：O(n^2)

  平均：O(n logn)

```java
    public static void quickSort(int[] arr, int low, int high) {
        if (arr.length < 1 || low < 0 || high >= arr.length || low >= high) {
            return;
        }
        int pivotPos = partition(arr, low, high);
        quickSort(arr, low, pivotPos - 1);
        quickSort(arr, pivotPos + 1, high);
    }

    public static int partition(int[] arr, int low, int high) {
        int pivot = arr[low];
        while (low < high) {
            while (low < high && arr[high] >= pivot) high--;
            arr[low] = arr[high];
            while (low < high && arr[low] <= pivot) low++;
            arr[high] = arr[low];
        }
        arr[low] = pivot;
        return low;
    }
```

## 7、堆排序

堆一般用数组来进行存储，已知堆中的某个节点在数组中的下标位置为 **`i`** ，那么它的父节点为 **`(i-1)/2`** ，左子节点为 **`2*i+1`** ，右子节点为 **`2*i+2`** 。

**大顶堆：`arr[i] >= arr[2i+1] && arr[i] >= arr[2i+2]`**  

**小顶堆：`arr[i] <= arr[2i+1] && arr[i] <= arr[2i+2]`**  

步骤：

- 将无序数组构建成一个大根堆，将堆顶最大元素与最后一个元素交换。
- 剩余元素继续重新构建大根堆，继续交换，最后会得到一个从小到大的排序的序列。（从大到小构建最小堆）

```java
class HeapSortTest {

    public static void heapSort(int[] arr) {
        buildHeap(arr);             // 根据当前数组构造大根堆
        int len = arr.length;
        while (len > 1) {
            swap(arr, 0, --len);  // 固定最大值
            heapify(arr, 0, len);   // 用剩下的数据继续构造大根堆
        }
    }

    public static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }

    public static void buildHeap(int[] arr) {       // 构造大根堆
        for (int i = 0; i < arr.length; i++) {
            int curIndex = i;                       // 当前位置的索引
            int fatherIndex = (curIndex - 1) / 2;   // 当前节点的父节点
            while (curIndex < fatherIndex) {
                swap(arr, curIndex, fatherIndex);   // 交换当前节点与父结点
                curIndex = fatherIndex;             // 将父节点设置为当前节点
                fatherIndex = (curIndex - 1) / 2;   // 重新计算当前节点的父节点
            }
        }
    }

    public static void heapify(int[] arr, int index, int len) {     // 将剩余的数继续构造成大根堆
        int left = 2 * index + 1;
        int right = 2 * index + 2;
        while (left < len) {
            // 从当前节点以及左右子节点三者中选出一个largestIndex
            int maxIndex = arr[right] > arr[left] && right < len ? right : left;
            maxIndex = arr[index] > arr[index] ? index : maxIndex;
            if (maxIndex == index) break;   // 如果当前父节点已经是最大节点了说明现在已经是大根堆了，结束循环
            swap(arr, maxIndex, index);     // 否则父节点和最大值交换
            index = maxIndex;               // 将当前位置指向交换后的较大值
            left = 2 * index + 1;
            right = 2 * index + 2;          // 更新左右子节点的值
        }
    }

    public static void main(String[] args) {
        int[] arr = {9, 8, 7, 6, 5, 4, 3, 2, 1};
        heapSort(arr);
        System.out.println(Arrays.toString(arr));
    }

}
```





# 海量数据如何实现排序？