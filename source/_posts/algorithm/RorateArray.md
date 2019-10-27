---
title: leetcode189-旋转数组
tags:
  - 数组
author: coderluo
cover: true
img: http://media.coderluo.top/blog/article/algorithm/189-封面.jpg
categories: algorithm
date: 2019-10-27 15:56:38
---




## 前言

2019.10.27日打卡

> 算法，即解决问题的方法。同一个问题，使用不同的算法，虽然得到的结果相同，但是耗费的时间和资源是不同的。这就需要我们学习算法，找出哪个算法更好。


## 题目

> 每天一道leetcode189. 旋转数组
> 分类： 数组
> 难度： 简单
> 题目链接: https://leetcode-cn.com/problems/rotate-array/


## 题目描述
> 给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。


**示例 1:**
> 输入: [1,2,3,4,5,6,7] 和 k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右旋转 1 步: [7,1,2,3,4,5,6]
向右旋转 2 步: [6,7,1,2,3,4,5]
向右旋转 3 步: [5,6,7,1,2,3,4]


**示例 2:**
> 输入: [-1,-100,3,99] 和 k = 2
输出: [3,99,-1,-100]
解释: 
向右旋转 1 步: [99,-1,-100,3]
向右旋转 2 步: [3,99,-1,-100]


**说明：**

- 尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。
- 要求使用空间复杂度为 O(1) 的 原地 算法。




## 题解

### 自己做


- **思路**

我太菜了，感觉原地移动的好复杂，只好使用额外内存来实现。即将数组放入列表中，由于列表可以对头尾进行操作，因为每向右移动一位，从尾部删除，将删除的元素在插入到头部即可实现。

- **代码实现**

```java
class Solution {
    public void rotate(int[] nums, int k) {
        ArrayList<Integer> list = new ArrayList<>();
        for(int i=0; i<nums.length; i++){
            list.add(nums[i]);
        }
        for(int i = 0; i < k; i++) {
            int target = list.remove(nums.length-1);
            list.add(0,target);
        }
        int[] array = new int[]{};
        for (int i =0;i<list.size();i++) {
            nums[i] = list.get(i);
        }
    }
}
```

代码比较简单，也没有什么需要具体解析的，这里使用了额外内存，将数组元素放到列表中。理应是不符合题目要求的，不过没办法，毕竟算法太菜了，下面会有最优解。

- **复杂度分析**

1. 时间复杂度：O(n+n+n) = O(n)。 `for` 循环的时间复杂度是 O(n),一共使用了三次for循环。
2. 空间复杂度：O(n)。List需要的空间跟nums中元素个数相等。



- **执行结果**


![](http://media.coderluo.top/blog/articleleetcode189-1.png)


### 参考解法

- **思路** 

本题提供下列两种思路：

1. 双重循环：
暴力，第一层循环为需要右移的个数，第二层循环移动所有元素值到正确的位置（这个需要反思，应该能想到的，不过不推荐）。
2. 翻转：
arr = [1,2,3,4,5] --右移两位--> [4,5,1,2,3] 
假设 n = arr.length，k = 右移位数，可得：  
    --[1,2,3,4,5] --翻转索引为[0,n-1]之间的元素--> [5,4,3,2,1] 
    --翻转索引为[0,k-1]之间的元素--> [4,5,3,2,1] 
    --翻转索引为[k,n-1]之间的元素--> [4,5,1,2,3]
                 
> 旋转数组其实就是把数组分成了两部分，解题关键就是在保证原有顺序的情况下
把后面一部分移到前面去。数组整体翻转满足了第二个要素，但是打乱了数组的
原有顺序，所以此时再次对两部分进行翻转，让他们恢复到原有顺序（翻转之后
再翻转，就恢复成原有顺序了）。


- **代码实现**

```java

    /**
     * 双重循环
     * 时间复杂度：O(kn)
     * 空间复杂度：O(1)
     */
    public void rotate_1(int[] nums, int k) {
        int n = nums.length;
        k %= n;
        for (int i = 0; i < k; i++) {
            int temp = nums[n - 1];
            for (int j = n - 1; j > 0; j--) {
                nums[j] = nums[j - 1];
            }
            nums[0] = temp;
        }
    }

    /**
     * 翻转
     * 时间复杂度：O(n)
     * 空间复杂度：O(1)
     */
    public void rotate_2(int[] nums, int k) {
        int n = nums.length;
        k %= n;
        reverse(nums, 0, n - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, n - 1);
    }


    private void reverse(int[] nums, int start, int end) {
        while (start < end) {
            int temp = nums[start];
            nums[start++] = nums[end];
            nums[end--] = temp;
        }
    }
```


- **算法复杂度分析**

双重for循环:
- 时间复杂度：O(n∗k)
- 空间复杂度：O(1)

翻转:
- 时间复杂度：O(n)
- 空间复杂度：O(1)


### 执行结果

翻转执行结果：
![](http://media.coderluo.top/blog/article/algorithm/leetcode189-2.png)


## 结束语

> 小伙伴们可能在看了翻转这种解法，都会感到非常的巧妙，是怎么想到的。这个我觉得不必过于纠结，本身我们联系算法题，就是一个刻意提高的过程，做的多了，自然而然也就有想法了。

想一起打卡进步的朋友们,欢迎关注笔者公众号: **爱上敲代码**, 会定期分享Java技术干活,让枯燥的技术游起来!

![扫码关注](http://media.coderluo.top/blog/article/wechar-erweima/最新关注引导.png)