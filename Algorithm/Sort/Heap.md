# 堆排序

原文：https://www.runoob.com/w3cnote/heap-sort.html



​        堆排序（heapsort）是指利用堆这种数据结构所设计的排序算法。**堆积是一个近似完全二叉树的结构**，并同时满足**堆积的性质：子节点的键值或索引总是小于（或者大于）它的父节点**。堆排序可以说是一种利用堆的概念来排序的**选择排序**。分为两种方法：

1. 大顶堆：每个节点的值都大于等于器子节点的值，在推排序算法中用于升序排列
2. 小顶堆：每个节点的值都小于等于器子节点的值，在推排序算法中用于降序排列

​        堆排序的平均时间复杂度为 $O(nlogn)$

## 1 算法步骤

1. 创建一个堆 $H[0 \cdots n-1]$
2. 把堆首（最大值）和堆尾互换
3. 把堆的尺寸缩小 $1$， 并调用 *shift_down(0)*，目的是把新的数组顶端数据调整到相应位置
4. 重复步骤2，直到堆的尺寸为 $1$

## 2 动图演示

![heapSort](./images/heapSort.gif)

![heapSort2](./images/Sorting_heapsort_anim.gif)

## 3 代码实现

### 3.1 Python

```python
def buildMaxHeap(arr):
    import math
    for i in range(math.floor(len(arr)/2), -1, -1):
        heapify(arr,i)
        
def heapify(arr, i):
    left = 2 * i + 1
    right = 2 * i + 2
    largest = i
    if left < arrLen and arr[left] > arr[largest]:
        largest = left
    if right < arrLen and arr[right] > arr[largest]:
        largest = right
        
    if largest != i:
        swap(arr, i, largest)
        heapify(arr, largest)
        
def swap(arr, i, j):
  arr[i], arr[j] = arr[j], arr[i]
  
def heapSort(arr):
  global arrLen
  arrLen = len(arr)
  buildMaxHeap(arr)
  for i in range(len(arr)-1, 0, -1):
    swap(arr, 0, i)
    arrLen -= 1
    heapify(arr,0)
  return arr
```

### 3.2 Java

```java
public class HeapSort implements IArraySort {
  @Override
  public int[] sort(int[] sourceArray) throws Exception {
    // 对 arr 进行拷贝，不改变参数内容
    int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
    int len = arr.length;
    buildMaxHeap(arr, len);
    
    for (int i = len -1; i>0; i--) {
      swap(arr, 0, i);
      len--;
      heapify(arr,0,len); // 构建堆
    }
    return arr;
  }
  
  // 构建最大的堆
  private void buildMaxHeap(int[] arr, int len) {
    for (int i=(int)Math.floor(len /2); i>=0;i--) {
      heapify(arr, i, len);
    }
  }
  
  private void heapify(int[] arr, int i, int len) {
    int left = 2 * i + 1;
    int right = 2 * 2 + 2;
    int largest = i;
    
    if (left < len && arr[left] > arr[largest]) {
      largest = left;
    }
    
    if (right < len && arr[right] > arr[largest]) {
      largest = right;
    }
    if (largest != i) {
      swap(arr, i, largest);
      heapify(arr, largest, len);
    }
  }
  
  private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
  }
}
```

