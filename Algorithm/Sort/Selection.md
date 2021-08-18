# 选择排序

原文：https://www.runoob.com/w3cnote/selection-sort.html

（https://www.runoob.com/w3cnote_genre/algorithm）



​        选择排序是一种比较简单直观的排序算法，无论什么数据进去，都是 $O(n^2)$ 的时间复杂度。所以用它的时候，数据规模越小越好。唯一的好处可能就是不占用额外的内存空间。

## 1 算法步骤

1. 首先，在未排序序列中找到最小（大）的元素，存放到排序序列的起始位置

2. 再从剩余未排序元素中继续寻找最小（大）的元素，然后放到已排序序列的末尾
3. 重复第二步，知道所有的元素均排序完毕。

## 2 动画演示

![selectionSort](./images/selectionSort.gif)

## 3 代码实现

### 3.1 Java实现

```java
public class SelectionSort implement IArraySort {
    @override
    public int[] sort(int[] sourceArray) throws Exception {
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
        
        // 总共要经过 n-1 轮比较
        for (int i=0; i<arr.length -1; i++) {
            int min = i;
            
            // 每轮需要比较的次数 n - i
            for(int j = i+1; j < arr.length; j++) {
                if (arr[j] < arr[min]) {
                    // 记录目前能找到最小元素的下标
                    min = j;
                }
            }
            
            // 将找到的最小值和 i 位置所在的值进行交换
            if (i != min) {
                int tmp = arr[i];
                arr[i] = arr[min];
                arr[min] = tmp;
            }
        }
        return arr;
    }
}
```

### 3.2 Python实现

```python
def selectionSort(arr):
    for i in range(len(arr) - 1):
        # 记录最小的索引
        minIndex = i
        for j in range(i+1, len(arr)):
            if arr[j] < arr[minIndex]:
                minIndex = j
        # i 不是最小数时，将 i 和最小数进行交换
        if i != minIndex:
            arr[i], arr[minIndex] = arr[minIndex], arr[i]
    return arr
```

