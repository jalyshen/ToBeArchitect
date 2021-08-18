# 冒泡排序

原文：https://www.runoob.com/w3cnote/bubble-sort.html



​        冒泡排序是一种简单直观的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误（如果定义“左边<右边”是正确的顺序，那么如果“左边>右边”，这个就是错误的顺序了）就把它们交换过来。走访数列的工作是重复地进行，直到没有再需要交换。也就是说，该数列已经派完序了。这个算法的名字由来，是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

​        冒泡排序还有一种优化算法，就是立一个flag，当在一趟序列遍历中元素没有发生交换，则证明该序列已经有序。但是这种该井并没有提升排序的性能。

​        冒泡排序的时间复杂度是：
$$
(n-1) + (n-2) + (n -3) + \cdots + 2 + 1 = n \times \frac{n-1}{2}
$$
即：$O(n^2)$



## 1 算法步骤

​        比较相邻的元素，如果第一个比第二个大，则交换它们两个。

​        对每一对相邻元素做同样的工作，从开始第一对到结尾的最后一对。这步做完后，最大的数就拍到了队列最后。

​        针对所有的元素重复以上的步骤，除了最后一个。

​        持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

## 2 动画演示

![bubble](./images/bubbleSort.gif)

## 3 什么时候最快

​        当输入的数据队列是已经排好序时

## 4 什么时候最慢

​        当输入的数据队列是反序时

## 5 Java 实现

```java
public class BubbleSort implement IArraySort {
    @override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);
        for (int i=1; i<arr.length; i++){
            // 设定一个标记，若为true，则表示此次循环没有进行交换，
            // 也就是待排序队列已经有序，排序完成。
            boolean flag = true;
            for (int j = 0; j < arr.lenght -i; j++) {
                if (arr[j] > arr[j + 1]) {
                    int tmp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = tmp;
                    
                    flag = false;
                }
            }
            if (flag) {
                break;
            }
        }
        return arr;
    }
}
```

## 6 Python实现

```python
def bubbleSort(arr):
    for i in range(1, len(arr)):
        for j in range(0, len(arr) - i):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j +1], arr[j]
    return arr
```

