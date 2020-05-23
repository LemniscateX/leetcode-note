# Longest Substring Without Repeating Characters
## 题目描述
Given a string, find the length of the longest **substring** without repeating characters.

## 输入输出示例

> Input: "abcabcbb"  
> Output: 3  
> Explanation: The answer is "abc", with the length of 3. 

> Input: "bbbbb"  
> Output: 1  
> Explanation: The answer is "b", with the length of 1.

> Input: "pwwkew"  
> Output: 3  
> Explanation: The answer is "wke", with the length of 3.  
>              Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

## 解题思路
### v1: Wrong Answer
暴力解法。

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
    var max = 0;
    for (var i = 0; i < s.length - 1; i++) {
        for (var j = i + 1; j < s.length; j++) {
            var left = s.slice(i, j);
            if (left.indexOf(s[j]) == -1) {
                max = Math.max(max, left.length + 1);
            } else {
                max = Math.max(max, left.length);
                break;
            }
        }
    }
    return max;
};
```

出错了，错误样例：

> Input:    " "  
> Output:   0  
> Expected: 1

又忘记处理特殊情况了。

### v2: Accepted

特殊情况有以下几种：

- 输入""，输出0
- 输入" "，输出1
- 输入"x"，输出1

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
    if (s.length == 0)
        return 0;
    var max = 1;
    for (var i = 0; i < s.length - 1; i++) {
        for (var j = i + 1; j < s.length; j++) {
            var left = s.slice(i, j);
            if (left.indexOf(s[j]) == -1) {
                max = Math.max(max, left.length + 1);
            } else {
                max = Math.max(max, left.length);
                break;
            }
        }
    }
    return max;
};
```

暴力方法太不友好啦，还是想想可以怎么优化吧。

### v3: Wrong Answer
开始以为考察的是写个表达式，然后从中间状态递推到其他状态，但是没想出来，后来看了几篇题解，发现考察点也不是这个...拿小本本记下来吧。

首先，特殊情况是字符串长度为0和1的情况，这种情况下直接返回它的长度就行。

那要如何求最长的连续且无重复字母的子串呢？

可以考虑拿两个指针在字符串上动态移动，逐步找到每个符合条件的子串，在通过`Math.max()`的计算来获取最大子串长度。两个指针记录当前符合要求串的起始和结束位置，结束位置随着循环一个个向后递增，起始位置则需要考虑当前字符是否已存在于前一个子串中。

但是如果对于每一个子串，都去用`indexOf()`之类的方法来考察是否已存在于子串中，这势必带来比较高的复杂度，这时候就又可以想到采用hash的方法来进行查找的优化。

具体来说，我们需要记录每个字符的出现位置，直接使用map即可。假设当前考察字符下标为`j`，字符值为`c`，子串起始下标为`i`，那么：

- `map[c]`为空，说明前面没有遇到`c`，可以放心地将它视为子串的一部分，因此起始点`i`无需移动，结束点`j`需要`+1`来考察下一个字符。继而看看这个操作能否更新最大值，并更新`map[c]`。

- `map[c]`非空，说明前面有遇到`c`，这时不一定可以将它视为子串的一部分，但是还是需要移掉前一个出现的`c`，这样才能去考察后面能否出现更长子串(如果不移除前一个则后面都无法加入子串来进行考察)，因此起始点`i`需要移到`map[c]`的下一个位置，结束节点`j`则依然`+1`来考察下一个字符。继而看看这个操作能否更新最大值，并更新`map[c]`。

写出代码如下：

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
    if (s.length < 2) {
        return s.length;
    }
    var i = 0, j = 0, len = s.length, map = {}, max = 0;;
    while (i < len && j < len) {
        if (map[s[j]] == undefined) {
            map[s[j]] = j++;
        } else {
            i = map[s[j]] + 1;
            map[s[j]] = j++;
        }
        max = Math.max(max, j - i);
    }
    return max;
};
```

举个例子：

> Input:  "pwwkew"  
> Output: 3

第一次：
- `map[p]`为空，将其加入map，map为`{p: 0}`
- `j++`来考察下一个
- `max= Math.max(0, 1-0)=1`(比较`""`和`"pw"`)

第二次：
- `map[w]`为空，将其加入map，map为`{p: 0, w: 1}`
- `j++`来考察下一个
- `max= Math.max(1, 2-0)=2`(比较`"p"`和`"pw"`)

第三次：
- `map[w]`非空，将`i`更新为上次`w`出现的下一位置即`2`，更新map为`{p: 0, w: 2}`
- `j++`来考察下一个
- `max= Math.max(2, 3-2)=2`(比较`"pw"`和`"w"`)

第四次：
- `map[k]`为空，将其加入map，map为`{p: 0, w: 2, k: 3}`
- `j++`来考察下一个
- `max= Math.max(2, 4-2)=2`(比较`"pw"`和`"wk"`)；

第五次：
- `map[e]`为空，将其加入map，map为`{p: 0, w: 2, k: 3, e: 4}`
- `j++`来考察下一个
- `max= Math.max(2, 5-2)=3`(比较`"pw"`/`"wk"`和`"wke"`)；

第六次：
- `map[w]`非空，将`i`更新为上次`w`出现的下一位置即`3`，更新map为`{p: 0, w: 5}`
- `j++`来考察下一个
- `max= Math.max(3, 6-3)=3`(比较`"wke"`和`"kew"`)。

差不多就是这个流程。
不过还是答案错误了，错误样例：

> Input:    "abba"  
> Output:   3  
> Expected: 2

### v4: Accepted
按照代码的流程写一下错误样例的过程，会发现，在处理到第二个`a`的时候会出现问题。这时使用规则`i = map[s[j]] + 1`来更新：

- 上一步的`i`：由于上一步考察的是`b`，考察前将`i`更新为了上次`map[b]`的下一位置即`2`

- 此时的`i`：被更新为上次`map[a]`的下一位置即`1`，这时的`max= Math.max(2, 4-1)=3`(比较`"ab"`和`"bba"`)。

也就是说，这时我们错误地设置了`i`的值，只关注当前考察字符的上一出现位置是不行的。通过观察上一步`i`的值我们可以发现，上一步的`i`值就是应该设置的值，因此尝试对其进行一定改动。

```javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
    if (s.length < 2) {
        return s.length;
    }
    var i = 0, j = 0, len = s.length, map = {}, max = 0;;
    while (i < len && j < len) {
        if (map[s[j]] == undefined) {
            map[s[j]] = j++;
        } else {
            i = Math.max(map[s[j]] + 1, i); // modified
            map[s[j]] = j++;
        }
        max = Math.max(max, j - i);
    }
    return max;
};
```
通过！

## 反思
- 对滑动窗口不太熟悉
- 对下标的加减不太敏感

## 标签
- 哈希表
- 滑动窗口
