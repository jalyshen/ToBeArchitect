# 希尔排序

原文：https://www.runoob.com/w3cnote/shell-sort.html



​        希尔排序，也称为递减增量排序算法，是插入排序的一种更高效的改进版本。但希尔排序是**非稳定排序算法**。

​        希尔排序是基于插入排序的一下两点性质而提出改进的：

* 插入排序在对几乎已经排好序的数据操作时，效率高，既可以达到线性排序的效率
* 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位

​        希尔排序的基本思想是：先将整个待排序的记录序列**分割成若干子序列**分别进行直接插入排序，待整个序列中的记录“**基本有序**”时，再对全体记录进行依次直接插入排序。

## 1 算法步骤

1. 选择一个增量序列$t_1, t_2, \cdots, t_k$，其中， $t_i > t_j, t_k = 1$
2. 按增量序列个数 $k$，对序列进行 $k$ 趟排序
3. 每趟排序，根据对应的增量 $t_i$，将待排序列分割成若干长度为 $m$ 的子序列，分别对各子表进行直接插入排序。仅增量因子为 $1$ 时，整个序列作为一个表来处理，表长度即为整个序列的长度

## 2 动画演示

![shellsort](./images/shellsort.gif)

## 3 代码实现

### 3.1 Java

```java
public class ShellSort {
    public static void shellSort(intp[] arr) {
        int length = arr.length;
        int temp;
        for (int step = length/2; step >= 1; step /= 2) {
            for (int i = step; i< length; i++) {
                temp = arr[i];
                int j = i = step;
                while( j>=0 && arr[j] > temp) {
                    arr[j + step] = arr[j];
                    j -= step;                    
                }
                arr[j + step] = temp;
            }
        }
    }
}
```



### 3.2 Python

```python
def shellSort(arr):
    import math
    gap = 1
    while(gap < len(arr/3)):
       gap = gap * 3 +1
    while gap > 0:
        for i in range(gap, len(arr)):
            temp = arr[i]
            j = i - gap
            while j >=0 and arr[j] > temp;
                arr[j+gap] = arr[j]
                j-= gap
            arr[j+gap] = temp
        gap = math.floor(gap/3)
    return arr
```

