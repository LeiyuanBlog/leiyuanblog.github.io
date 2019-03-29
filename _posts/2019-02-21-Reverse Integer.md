---
layout:     post
title:      Reverse Integer
subtitle:   Reverse Integer
date:       2019-02-21
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 算法分析
---

> 在这里小小推荐下我的个人博客
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)
>

#### 前言

1. 最近准备把算法慢慢的捡起来，所以准备日更一道算法题目，难度自然是由简入难，所以同学们可以每天都来看看小编的更新。
2. 北京的生活总是匆忙的，希望在北京的活着将要来北京生活的伙伴们，不要被这无情的生活淹没。最起码抽出一些时间，来做一些自己喜欢的事情。

#### 给定32位有符号整数，整数的反向数字。

**例1：**

```
输入： 123
 输出： 321
```

**例2：**

```
输入： -123
 输出： -321
```

**例3：**

```
输入： 120
 输出： 21
```

**注意：**
假设我们正在处理的环境中，其只能在32位带符号整数的范围内存储的整数：[-2147483648，2147483647] 出于此问题的目的，假设当反向整数溢出时，函数返回0。

### 解题思路

1. 可以想到的是可以通过字符串来进行反转是一种非常简单的方法。

2. 可以通过截取字符串的第一个字符进行区别正负。

3. 可以通过异常处理来判断是否超过取值范围。

4. 代码实现：

   ```java
   class Solution {
       public int reverse(int x) {
           // 将数字转换成字符串
           String str = x + "";
           // 定义变量答案字符
           StringBuffer answer = new StringBuffer();
           // 判断第一个字符是不是“-”号
           if ("-".equals(str.charAt(0) + "")) {
               // 将“-”号添加到answer末尾
               answer.append("-");
               // 截取“-”号后所有数字
               str = str.substring(1);
           }
           StringBuffer stringBuffer = new StringBuffer(str);
           // 反转数字
           stringBuffer = stringBuffer.reverse();
           // 将反转后的数字添加到answer后
           answer.append(stringBuffer);
           // 通过异常处理判断是否超出取值范围
           try {
               // 将字符串强转为数字，若穿出范围则抛出异常
               return Integer.parseInt(answer.toString());
           } catch (Exception e) {
               // 超出异常则返回0
               return 0;
           }
       }
   }
   ```

**最后说两句**

1. 所有的题目都有很多种解法，我的一定不是最好的，甚至可以说是比较低端的解法，希望大牛们多多指教！！！
2. 如果朋友们对算法、编程有很大兴趣的话，可以私信我，大家一同探讨；相互学习、共同进步。
3. 朋友们如果对这道题目有更好的解法，希望可以在评论中指出，让大家一起讨论学习。
4. 最后感谢大家的阅读以及关注，谢谢大家！！！