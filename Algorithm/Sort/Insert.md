# 插入排序

原文：https://www.runoob.com/w3cnote/insertion-sort.html



​        插入排序的代码实现虽然没有冒泡排序和选择排序那么简单粗暴，但它的原理应该是最容易理解的了，因为只要打过扑克牌的人都应该能够秒懂。插入排序是一种最简单直观的排序算法，它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

​        插入排序和冒泡排序一样，也有一种优化算法，叫做拆半插入。

## 1 算法步骤

1. 将第一待排序序列的第一个元素看作一个有序序列，把第二个元素到最后一个元素当作时未排序序列
2. 从头到尾，依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置（如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面）

## 2 动图演示

![insertSort](./images/insertionSort.gif)

## 3 代码实现

### 3.1 Java

```java
public class InsertSort implements IArraySort {
    @override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
        
        // 从下标为 1 的元素开始，选择合适的位置插入，
        // 因为下标为 0 的元素只有一个元素，默认时有序的
        for (int i = 1; i < arr.length; i++) {
            // 记录要插入的数据
            int tmp = arr[i];
            
            // 从已经排序的序列最右边的开始比较，找到比其小的数
            int j = i;
            while (j > 0 && tmp < arr[j - 1]) {
                arr[j] = arr[j-1];
                j--;
            }
            // 存在比其小的数，插入
            if (j != i) {
                arr[j] = tmp;
            }
        }
        return arr;
    }
}
```



### 3.2 Python

```python
def insertSort(arr):
    for i in range(len(arr)):
         preIndex = i-1
         current = arr[i]
         while preIndex >=0 and arr[preIndex] > current:
             arr[preIndex + 1] = arr[preIndex]
             preIndex-=1
         arr[preIndex + 1] = current
     return arr
```

