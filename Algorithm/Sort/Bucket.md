# 桶排序

原文：https://www.runoob.com/w3cnote/bucket-sort.html



## 1 算法步骤

​        桶排序时记数排序的升级版。它利用了函数的映射关系，高效与否的关键就在于这个映射函数的确定。为了使桶排序更高效，需要做到以下两点：

* 再额外空间充足的情况下，尽量增大桶的数量
* 使用的映射函数能够将输入的 $N$ 个数据均匀的分配到 $K$ 个桶中

同时，对于桶中元素的排序，选择何种比较排序算法对于性能的影响至关重要。

### 1.1 什么时候最快

​        当输入的数据可以均匀的分配到每一个桶中

### 1.2 什么时候最慢

​        当输入的数据被分配到同一个桶中

## 2 示意图

​        元素分配在桶中：

![1](./images/Bucket_sort_1.svg_.png)

​        然后，元素在每个桶中排序：

![2](./images/Bucket_sort_2.svg_.png)

## 3 代码实现

### 3.1. Java

```java
public class BucketSort implements IArraySort {
    private static final InsertSort insertSort = new InsertSort();
    
    @Override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
        
        return bucketSort(arr, 5);
    }
    
    private int[] bucketSort(int[] arr, int bucketSize) throws Exception {
        if (arr.length == 0) {
            return arr;
        }
        int minValue = arr[0];
        int maxValue = arr[0];
        for (int value : arr) {
            if (value < minValue) {
                minValue = value;
            } else if (value > maxValue) {
                maxValue = value;
            }
        }
        
        int bucketCount = (int) Math.floor((maxValue - minValue) / bucketSize) + 1;
        int[][] buckets = new int[bucketCount][0];
        
        // 利用映射函数将数据分配到各个桶中
        for (int i = 0; i < arr.length; i++ ){
            int index = (int) Math.floor((arr[i] - minValue) / bucketSize);
            buckets[index] = arrAppend(buckets[index].arr[i]);
        }
        
        int arrIndex = 0;
        for (int[] bucket : buckets) {
            if (bucket.length <= 0) {
                continue;
            }
            // 对每个桶进行排序，这里使用了插入排序
            bucket = insertSort.sort(bucket);
            for (int value: bucket) {
                arr[arrIndex++] = value;
            }
        }
        return arr;
    }
    
    private int[] arrAppend(int[] arr, int value) {
        arr = Arrays.copyOf(arr, arr.length + 1);
        arr[arr.length - 1] = value;
        return arr;
    }
}
```