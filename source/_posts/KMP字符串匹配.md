---
title: KMP字符串匹配
excerpt: KMP字符串匹配算法分析
date: 2022-03-02 21:36:46
tags:
 - KMP
categories:
 - 算法
---

# 题目
给你两个字符串`haystack`和`needle`，请你在`haystack`字符串中找出`needle`字符串出现的第一个位置（下标从 0 开始）。如果不存在，则返回`-1`
**说明：**
当`needle`是空字符串时应当返回0，与C语言中的`strstr()`以及Java中的`indexOf()`定义相符

- 示例1：
{% codeblock %}
	输入：haystack = "hello", needle = "ll"
	输出：2
{% endcodeblock %}

- 示例2：
{% codeblock %}
	输入：haystack = "aaaaa", needle = "bba"
	输出：-1
{% endcodeblock %}

- 示例3：
{% codeblock %}
	输入：haystack = "", needle = ""
	输出：0
{% endcodeblock %}

# 思路+代码
本题可以采用暴力解法完成字符串匹配，但是在LeetCode中提交会超时，需要采用KMP算法完成此题

## KMP理论
Knuth–Morris–Pratt（KMP）算法是一种改进的字符串匹配算法，它的核心是利用匹配失败后的信息，尽量减少模式串与主串的匹配次数以达到快速匹配的目的。它的时间复杂度是***O(m + n)***

## 构建next数组
在完成KMP算法之前，需要构建***next数组***。`next[i]`所对应的含义为：`P[0, 1, ..., i-1]`的最长公共前缀后缀的长度，令`p[0] = -1`
例如字符串`abcba`:
- 前缀包括：`a, ab, abc, abcb`
- 后缀包括：`bcba, cba, ba, a`
- 最长公共前缀后缀：`a`，长度为`1`
图解如下：
|         |    A    |    C    |    T    |    G    |    P    |    A    |    C    |    Y    |
|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|:-------:|
|   next  |   -1    |    0    |    0    |    0    |    0    |    0    | ***1*** | ***2*** |

{% codeblock %}
	vector<int> gextNext(string needle)
    {
        int length = needle.size();
        vector<int> N(length);
        N[0] = -1;
        int k = -1;
        int j = 0;
        while(j < length - 1)
        {
            if (k < 0 || needle[k] == needle[j])
            {
                k++;
                j++;
                N[j] = k;
            }
            else
            {
                k = N[k];
            }
        }
        return N;
    }
{% endcodeblock %}

## KMP思路
- 当主串与子串的数组索引分别停留在`i`与`j`
- 发现此时两个位置的字符不匹配，基于`next`数组将子串的索引更新到`next[j]`
- 此时主串的索引不动，与更新后的子串索引所在位置进行比较

{% codeblock %}
	int strStr(string haystack, string needle) {
    int haystack_length = haystack.size();
    int needle_length = needle.size();
    if (needle_length == 0)
        return 0;

    vector<int> next = gextNext(needle);
    int h_index = 0;
    int n_index = 0;
    while((h_index < haystack_length) && (n_index < needle_length))
    {
        if (n_index < 0 || (haystack[h_index] == needle[n_index]))
        {
            h_index++;
            n_index++;
        }
        else
        {
            n_index = next[n_index];
        }
    }

    if (n_index == needle_length)
    {
        return h_index - n_index;
    }
    else
    {
        return -1;
    }
}
{% endcodeblock %}

# 辅助理解资料

## 递推求next数组
我们很容易的可以知道，`next[0] = -1`，`next[1] = 0`也是容易推得的。那么当`j > 1`时，如果我们已知了`next[j]`，那么`next[j + 1]`怎么求得呢？？？
下面分两种情况：
- 当`P[K] = P[j]`时，`next[j+1] = next[j] + 1 = k + 1`，当前模式串中在`j + 1`所对应字符前有`K + 1`长度的最大公共前后缀
- 当`P[K] != P[j]`时，说明`P[0]P[1]...P[k-1]P[k] != P[j-k]P[j-k+1]...P[j]`，也就是当前模式串中在`j + 1`所对应字符前没有长度为`K + 1`的最大公共前后缀，只能寻找更短的最大公共前后缀
- 因此，在`P[0]P[1]...P[k-1]P[k]`中不断递归索引`k = next[k]`，找到一个字符`P[K']`，那么最大公共前后缀长度就是`K' + 1`S

## 解释k = next[k]能找到长度更短的最大公共前后缀
![](https://cdn.jsdelivr.net/gh/MinghuiJia/CDN-source/KMP_String_Match/KMP1.png)

<br>
<br>


来源：LeetCode、CSDN
链接：[实现strStr()](https://leetcode-cn.com/leetbook/read/array-and-string/cm5e2/)、[KMP算法详解](https://blog.csdn.net/yyzsir/article/details/89462339)
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
