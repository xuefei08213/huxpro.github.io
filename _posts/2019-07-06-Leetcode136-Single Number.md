---
layout:     post
title:      "Leetcode136-Single Number"
subtitle:   ""
date:       2019-07-01 12:00:00
author:     "xuefei"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - leetcode
---


## 题目内容

给定一个非空整型数组，其中只有一个整数出现次数为1，其他的出现次数都为2。要求找出只出现一次的整数。

## 注意点
实现算法需要是线性的时间复杂度，并且不需要额外的内存。

## 示例1
输入：[2,2,1]  
输出：1

## 示例2
输入：[4,1,2,1,2]  
输出：4

### 使用HashMap解题
+ 遍历数组
+ 如果HashMap中已经存在对应的数值，则从HashMap中移除
+ 如果HashMap中不存在对应的数值，则加入HashMap中

```java
    public int solutionWithMap(int[] nums) {

        Map<Integer, Integer> map = new HashMap<>();
        for (int num : nums) {
            if (map.containsKey(num)) {
                map.remove(num);
            } else {
                map.put(num, num);
            }
        }

        return map.values().iterator().next();

    }
```
#### 算法解析
- 时间复杂度：O(n)。遍历数组的时间复杂都为O(n)，操作HashMap为O(1)
- 空间复杂度：O(n)。应为HashMap需要跟数组元素数量来存储元素

### 使用位操作解题

上一种解题方式是通过在内存中，找到出现次数为1的数值。除此还可以通过XOR位运算符直接进行位操作来算出只出现一次的数值。

XOR的运算规则如下：

|  a  |   b |  a ^ b |
|---|---|---|
|  0  |  0  |  0  |
|  0  |  1  |  1  |
|  1  |  0  |  1  |
|  1  |  1  |  0  |

两个相同的数值通过XOR位运算之后值为0，0与任意数XOR操作为任意数本身

      0010 = 2
    ^ 0010 = 2
    ======
      0000 = 0
    ^ 0001 = 1
    ======
      0001 = 1

根据上面的结论，只要遍历数组，并将每个数值执行XOR操作即可，代码如下

```java
    public int solutionXOR(int[] nums) {
        
        int result = 0;
        for (int num : nums) {
            result ^= num;
        }
        return result;
    }
```

#### 算法解析
- 时间复杂度：O(n)。遍历数组的时间复杂都为O(n)。
- 空间复杂度：O(1)。






