# 计数排序

原文：https://www.runoob.com/w3cnote/counting-sort.html

​    

​        计数排序的核心，在于将输入的数据值转换为健存储在额外开辟的数组空间中。作为一种线性时间复杂度的排序，**计数排序要求输入的数据必须使有确定范围的整数**。

## 1 算法步骤

​        当输入的元素是 $N$ 个 $0$ 到 $K$ 之间的整数时，它的运行时间时 $O(N + K)$。计数排序不是比较排序，排序的速度快于任何比较排序算法。

​        由于用来计数的数组 $C$ 的长度取决于待排序数组中数据的范围（等于待排序数组的最大值与最小值的差加上$1$ ），这使得计数排序对于数据范围很大的数组，需要大量时间和内存。例如：计数排序时用来排序 $0$ 到 $100$ 之间的数字的最好的算法，但是，计数排序可以用在计数排序中的算法来排序数据范围很大的数组。

​        通俗的理解，例如有 $10$ 个年龄不同的人，统计出有 $8$ 个人的年龄比 A 小，那么 A 的年龄就排在第 $9$ 位，用这个方法可以得到其他每个人的位置，也就排好序了。当然，年龄重复时需要特殊处理（保证稳定性），这就是为什么最后要反向填充目标数组，以及将每个数字的统计减去 $1$ 的原因。

​        算法步骤如下：

* 找出待排序的数组中最大和最小的元素
* 统计数组中每个值为 $i$ 的元素出现的次数，存入数组 $C$ 的第 $i$ 项
* 对所有的计数累加（从 $C$ 中的第一个元素开始，每一项和前一项相加）
* 反向填充目标数组：将每个元素 $i$ 放在新数组的第 $C_i$ 项，每放一个元素就将 $C_i$ 减去$1$

## 2 动画演示

![1](./images/countingSort.gif)

## 3 代码实现

### 3.1 Python

```python
def countingSort(arr, maxValue):
    bucketLen = maxValue + 1
    bucket = [0] * bucketLen
    sortedIndex = 0
    arrLen = len(arr)
    for i in range(arrLen):
        if not bucket[arr[i]]:
            bucket[arr[i]] = 0
        bucket[arr[i]]+=1
    for j in range(bucketLen):
        while bucket[j] > 0:
            arr[sortedIndex] = j
            sortedIndex+=1
            bucket[j]-=1
    return arr
```

### 3.2 Java

```java
public class CountingSort implements IArraySort {
    @Override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
        
        int maxValue = getMaxValue(arr);
        return countingSort(arr, maxValue);
    }
    
    private int[] countingSort(int[] arr, int maxValue) {
        int bucketLen = maxValue + 1;
        int[] bucket = new int[bucketLen];
        
        for (int value : arr){
            bucket[value]++;
        }
        
        int sortedIndex = 0;
        for (int j = 0; j < bucketLen; j++) {
            while (bucket[j] > 0) {
                arr[sortedIndex++] = j;
                bucket[j]--;
            }
        }
        return arr;
    }
    
    private int getMaxValue(int[] arr) {
        int maxValue = arr[0];
        for (int value: arr) {
            if (maxValue < value) {
                maxValue = value
            }
        }
        return maxValue;
    }
}
```

