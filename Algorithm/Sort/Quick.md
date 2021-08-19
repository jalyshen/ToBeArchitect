# 快速排序

原文：https://www.runoob.com/w3cnote/quick-sort-2.html



​        在平均状况下，排序 $n$ 个项目要 $O(nlogn)$ 次比较。在最坏的情况下，需要 $O(n^2)$ 次比较，但这种状况并不常见。事实上，快速排序通常明显比其他 $O(nlogn)$ 算法更快，因为它的内循环（inner loop）可以在大部分的架构上很有效的被实现出来。

​        快速排序又是一种分而治之的思想在排序算法上的典型应用。本质上，**快速排序**

**应该算是在冒泡的基础上的递归分治法**。

## 1 算法步骤

1. 从数列中挑出一个元素，称为“基准”（pivot）
2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆放在基准的后面（相同的数可以到任意一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partiton）操作
3. 递归的（recursive）把小于基准只元素的子数列和大于基准值元素的子数列排序

## 2 动图演示

![quickSort](./images/quickSort.gif)

## 3 代码实现

### 3.1 Java

```java
public class QuickSort implements IArraySort {

    @Override
    public int[] sort(int[] sourceArray) throws Exception {
    	// 对 arr 进行拷贝，不改变参数内容
    	int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
    	return quickSort(arr, 0, arr.length - 1);
    }
    
    private int[] quickSort(int[] arr, int left, int right) {
        if (left < right) {
            int partitionIndex = partition(arr, left, right);
            quickSort(arr, left, partitionIndex - 1);
            quickSort(arr, partitionIndex + 1, right);
        }
        return arr;
    }
    
    // 获取分区的位置 
    private int partition(int[] arr, int left, int right) {
        // 设定基准值(pivot)
        int pivot = left;
        int index = pivot + 1;
        for (int i = index; i<= right; i++) {
            if (arr[i] < arr[pivot]) {
                swap(arr, i, index);
                index++;
            }
        }
        swap(arr, pivot, index -1);
        return index -1;
    } 
    
    private void swap(int[] arr, int i, int j) {
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```



### 3.2 Python

```python
def quickSort(arr, left=None, right=None):
    left = 0 if not isinstance(left, (int, float)) else left
    right = len(arr)-1 if not isinstance(right,(int,float)) else right
    if left < right:
        partitionIndex = partition(arr, left, right)
        quickSort(arr, left, partitionIndex - 1)
        quickSort(arr, partitionIndex + 1, right)
    return arr

def partition(arr, left, right):
    pivot = left
    index = pivot + 1
    i = index
    while i <= right:
        if arr[i] < arr[pivot]:
            swap(arr, i, index)
            index +=1
        i+=1
    swap(arr, pivot, index-1)
    return index-1
    
def swap(arr, i, j):
    arr[i], arr[j] = arr[j], arr[i]
```

