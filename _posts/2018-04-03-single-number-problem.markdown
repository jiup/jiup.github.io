---
layout:     post
title:      "Single Number I / II / III"
subtitle:   "LeetCode 136 / 137 / 260"
date:       2018-04-03 00:44:00
author:     "Jiupeng Zhang"
header-img: "img/in-post/coding.jpg"
tags:
    - LeetCode
    - Algorithm
---
题目出处: [Single Number](https://leetcode.com/problems/single-number/), [Single Number II](https://leetcode.com/problems/single-number-ii/), [Single Number III](https://leetcode.com/problems/single-number-iii/)

# Single Number
> Given an array of integers, every element appears twice except for one. Find that single one.<br><br>
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

题目要求在线性时间复杂度内找到数组中唯一不成对元素。
初看题目，一个比较符合直觉的做法，就是在遍历数组的同时用Set或Map做筛选，于是有了最基本的，

**解法一**：
```java
class Solution {
    public int singleNumber(int[] nums) {
        Set<Integer> buf = new HashSet<>(); // 由于该题不需要计数，所以我们在这里使用Set
        for (int i : nums) {
            if (buf.contains(i)) {
                buf.remove(i); // 移除数组中出现过的元素
            } else {
                buf.add(i); // 将首次出现的元素添加到集合
            }
        }
        assert buf.size() == 1; // 此时集合中只剩下那个唯一不成对的元素
        return buf.iterator().next(); // 返回该元素
    }
}
```
虽然提交顺利通过，但这显然不是最优解，因为题目中有提到`implement it without using extra memory`，而解法一中使用了Set，所以我们继续寻找其它的思路。

位运算常常可以得到非常简洁的代码。基于异或的性质`a ⊕ b = b ⊕ a`，`a ⊕ a = 0`，`0 ⊕ b = b`，通过对数组中所有元素叠加异或运算，我们可以顺利地抵消数组中成对的元素，从而得到那个唯一非成对元素。

**解法二**：
```java
class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for (int i : nums) {
            result ^= i; // 对数组中每一个元素叠加异或运算
        }
        return result; // 剩下的即不成对的元素
    }
}
```



# Single Number II
> Given an array of integers, every element appears *three* times except for one, which appears exactly once. Find that single one.<br><br>
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

这道题定义“成对元素”出现的次数由两次变成了三次。（为了避免误会，这里我们把满足上述条件的元素改名为“普通元素”，对应的“非成对元素”我们改称之为“异常元素”）

**解法一**：

解决这道题的一种思路，是对数组中每个元素的每一位出现1的次数进行记录，随后将计数的每一位对3取模(相当于问题一中抵消普通元素的过程)，这样得到的结果则是那个异常元素的值。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int[] count = new int[32]; // 记录每位出现1的次数
        for (int num : nums) {
            for (int i = 0; i < 32; i++) {
                count[i] += (num >> i) & 1;
            }
        }

        int result = 0; // 按位累计异常元素的值
        for (int i = 0; i < 32; i++) {
            result |= (count[i] % 3) << i;
        }
        return result;
    }
}
```

以上方法能够解决问题，但不够优雅，因为代码中使用了额外的内存，笔者下面介绍一下在LeetCode上习得的佛系解法。

**解法二**：

由于我们需要在不使用额外内存的前提下通过某种运算达到普通元素相互抵消的效果，融合上述计数取模的思想，我们考虑引入按位循环计数的方法解题。从当前角度出发，我们可以这样理解，题目一中result的作用其实相当于一个按位0-1循环计数器，记录了数组中所有元素对应位为1的次数，最终叠加运算的结果中，出现了1（实际是取模后为1）的位，也就是那个非成对元素的bit，因此，叠加运算后的result即为所求的异常元素。

![](http://7xp1jv.com1.z0.glb.clouddn.com/18-4-2/75539079.jpg?imageView2/0/w/150/)

为了将上述思路推广到当前题目中，我们需要一个0-1-2循环计数器。因此，我们需要为待计数元素的每一位至少提供两个bit来保持计数（2个bits理论上可以保持4个计数，但我们只使用其中的3个），也就是说，我们的计数器实际有`00 -> 01 -> 10`三个计数项，因此我们引入了两个变量twice和once分别用来保持高位和低位的计数`0 -> 0 -> 1`和`0 -> 1 -> 0`。

![](http://7xp1jv.com1.z0.glb.clouddn.com/18-4-2/12839558.jpg?imageView2/0/w/150/)

首先画出状态转移真值表：

| 当前 twice | 当前 once |   input   | 新的 twice | 新的 once | 实际计数  |
| :---:     | :---:     | :---:     | :---:     | :---:     | :---:     |
| 0         | 0         | 0         | 0         | 0         | 0         |
| 0         | 1         | 0         | 0         | 1         | 1         |
| 1         | 0         | 0         | 1         | 0         | 2         |
| 0         | 0         | 1         | 0         | 1         | 1         |
| 0         | 1         | 1         | 1         | 0         | 2         |
| 1         | 0         | 1         | 0         | 0         | 0 (%3)    |

分别算出twice和once的逻辑表达式：

$$\begin{align*}twice&=twice\cdot\overline{once}\cdot\overline{input}+\overline{twice}\cdot{once}\cdot{input}\end{align*}$$

$$\begin{align*}once&=\overline{twice}\cdot{once}\cdot\overline{input}+\overline{twice}\cdot\overline{once}\cdot{input}\\&=\overline{twice}\cdot({once}\cdot\overline{input}+\overline{once}\cdot{input})\\&=\overline{twice}\cdot(once\oplus{input})\end{align*}$$

得到了逻辑表达式之后，剩下的工作就很简单了，写代码咯。
```java
class Solution {
    public int singleNumber(int[] nums) {
        int twice = 0, once = 0;
        for (int i : nums) {
            int tmp = ~twice & (once ^ i); // once和twice需要同步更新 (1)
            twice = (twice & ~once & ~i) | (~twice & once & i);
            once = tmp;
        }
        return once;
    }
}
```
代码的确简洁了不少，但是行(1)还是有些多余，为了使once和twice同步更新，我们缓存了新的once，待twice更新后才覆盖原once值。为了省略缓存once值的过程，我们可以进一步简化twice的真值表：

| 当前 twice | 新的 once |   input   | 新的 twice |
| :---:     | :---:     | :---:     | :---:     |
| 0         | 0         | 0         | 0         |
| 0         | 1         | 0         | 0         |
| 1         | 0         | 0         | 1         |
| 0         | 1         | 1         | 0         |
| 0         | 0         | 1         | 1         |
| 1         | 0         | 1         | 0         |

得到新的`twice`逻辑表达式：

$$\begin{align*}twice&=twice\cdot\overline{once}\cdot\overline{input}+\overline{twice}\cdot\overline{once}\cdot{input}\\&=\overline{once}\cdot(twice\cdot\overline{input}+\overline{twice}\cdot{input})\\&=\overline{once}\cdot(twice\oplus{input})\end{align*}$$

更新代码。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int twice = 0, once = 0;
        for (int i : nums) {
            once = ~twice & (once ^ i);
            twice = ~once & (twice ^ i); // 新的表达式
        }
        return once;
    }
}
```
恩恩，这下终于满意了，看来写出佛系代码着实需要一番修行呢哈哈。


# Single Number III
> Given an array of numbers `nums`, in which exactly two elements appear only once and all the other elements appear exactly twice. Find the two elements that appear only once.<br><br>
> For example:<br>
> Given `nums = [1, 2, 1, 3, 2, 5]`, return `[3, 5]`.<br><br>
> 1.The order of the result is not important.<br>
> 2.Your algorithm should run in linear runtime complexity. Could you implement it using only constant space complexity?

和题目一不同的是，这里的数组中出现了两个非成对元素，如果照搬题目一中异或的思路，我们最终只能得到两个非成对元素异或后的结果，却没办法恢复出其中某个元素的值。

让我们先再思考一下异或运算的特点，这里可以参考我[之前的文章](/2015/08/12/xor-swap-numbers/)。如果按照题目一的代码运行本题的话，得到的结果中的每一位，实际上都保留了两个非成对元素在该bit上的异同关系（1或0）。换个角度说，我们可以通过选择该异或结果中任意一个值为1的位，就能把这两个非成对元素划分到不同的组中。这样，每个分组中就只存在一个非成对元素了（而且原来成对的元素也不会被分开到不同的组去，所以也不影响其自然抵消），问题三被分解成了两个问题一，遂搞定。
```java
class Solution {
    public int[] singleNumber(int[] nums) {
        int tmp = 0; // 先求出两个非成对元素异或后的结果
        for (int i : nums) {
            tmp ^= i;
        }

        int mask = 1; // 选择异或结果中最右侧值为1的位
        while ((mask & tmp) == 0) {
            mask <<= 1;
        }

        int[] result = new int[2];
        for (int i : nums) {
            // 根据mask将当前元素划分到两个组
            if ((i & mask) != 0) {
                result[0] ^= i; // 异或结果末位为1的组
            } else {
                result[1] ^= i; // 异或结果末位为0的组
            }
        }
        return result;
    }
}
```
这里还需要注意一个细节，就是如何任意挑选首次异或结果中一个值为1的位？

为了简化代码，笔者选择了以最右面的值为1的位做划分，最终提交顺利通过。后来笔者在评论区里发现了一段十分优雅的代码，仅通过一次运算即可求出分组的mask：

```java
int mask = tmp & ~(tmp - 1); // 亦可简化为 mask = tmp & -tmp;
```
假设我们首次异或的结果为`0110`，减1后取反码为`1010`，与前者做与运算得到的值，即为分组mask`0010`，真是巧妙！

**最终的代码**：

```java
class Solution {
    public int[] singleNumber(int[] nums) {
        int tmp = 0; // 先求出两个非成对元素异或后的结果
        for (int i : nums) {
            tmp ^= i;
        }

        tmp &= -tmp; // 求出分组的mask

        int[] result = new int[2];
        for (int i : nums) {
            // 如果我们只计算出其中一个组里的非成对元素，另一个通过二次异或求出亦可
            if ((i & tmp) != 0) {
                result[0] ^= i; // 异或结果末位为1的组
            } else {
                result[1] ^= i; // 异或结果末位为0的组
            }
        }
        return result;
    }
}
```
