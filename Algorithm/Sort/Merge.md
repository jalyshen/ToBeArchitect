# 归并排序

原文：https://www.runoob.com/w3cnote/merge-sort.html



​        归并排序（Merge Sort）是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。

​        作为典型的分而治之思想的算法应用，归并排序的实现有两种方法：

1. 自上而下的递归（所有递归的方法都可以用迭代重写，所有就有第二种方法）
2. 自下而上的迭代

## 1 算法步骤

1. 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
3. 比较两个指针指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
4. 重复步骤3，直到某一指针达到序列尾
5. 将另一序列剩下的所有元素直接复制到合并序列尾

## 2 动图演示

![mergesort](./images/mergeSort.gif)

## 3 代码实现

### 3.1 Java

```java
public class MergeSort implements IArraySort {

    @Override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
        
        if (arr.length < 2){
            return arr;
        }
        int middle = (int) Math.floor(arr.length / 2);
        
        int[] left = Arrays.copyOfRange(arr, 0, middle);
        int[] right = Arrays.copyOfRange(arr, middle, arr.length);
        
        return merge(sort(left), sort(right));
    }
    
    protected int[] merge(int[] left, int[] right) {
        int[] result = new int[left.length + right.length];
        int i = 0;
        while(left.length > 0 && right.length > 0) {
            if (left[0] <= right[0]) {
                result[i++] = left[0];
                left = Arrays.copyOfRange(left, 1, left.length);
            } else {
               result[i++] = right[0];
               right = Arrays.copyOfRange(right, 1, right.length);
            }
        }
        while (left.length > 0) {
            result[i++] = left[0];
            left = Arrays.copyOfRange(left, 1, left.length);
        }
        while(right.length > 0) {
            result[i++] = right[0];
            right = Arrays.copyOfRange(right, 1, right.length);
        }
        
        return result;
    }
}
```



### 3.2 Python

```python
def mergeSort(arr):
    import math
    if (len(arr) < 2):
        return arr
    middle = math.floor(len(arr)/2)
    left, right = arr[0:middle], arr[middle:]
    return merge(mergeSort(left), mergeSort(right))
    
def merge(left, right):
    return = []
    while left and right:
        if left[0] <= right[0]:
            result.append[left.pop(0)]
        else:
            result.append(right.pop(0))
    while left:
        result.append(left.pop(0))
    while right:
        result.append(right.pop(0))
    return result
```

