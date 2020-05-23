# Add Two Numbers
## 题目描述
You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.  
You may assume the two numbers do not contain any leading zero, except the number 0 itself.

## 输入输出示例

> Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)  
> Output: 7 -> 0 -> 8  
> Explanation: 342 + 465 = 807.

## 解题思路
### v1: Error
首先读懂题意，发现其实就是两个数值相加，只不过表示形式为倒着的链表，不过这恰好与我们日常计算习惯一致，挺好的。需要考虑以下几点：

- 进位处理  
模拟进行大数相加的过程中，需要对进位进行一些操作。由于进位的`carry`是需要在整个迭代过程中传递的，因此可以设置一个全局的`carry`并初始化为0。

- 每一位的加法处理  
对于每一位上的加法，它的值为两个数与(之前的)进位三者之和，需保留部分为个位，需进位部分为十位。

- 边界处理  
首先可以想一想，什么时候开始无法用上面的公式进行计算？
当其中一个链表被遍历完时，由于还有可能存在`carry`，所以后面还是需要相加的，加法步骤和上面一致(不过需要考虑后面补0的问题)，因此不用它作为循环结束条件；
当其中长的链表被遍历完时，除了`carry`需要加上，没有其他加数了，因此可以将它作为循环结束条件。

- 答案初始化  
结果链表需要一个头节点head(用于返回)和一个移动指针p(用于添加节点)，首先想到初始化`head = null`、`p = head`。

理清以上几个方面就可以开始写代码了：

```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function (l1, l2) {
    var p1 = l1, p2 = l2, carry = 0, lresHead = null, lresPointer = lresHead;
    while (p1 != null || p2 != null) {
        var left = (p1.val + p2.val + carry) % 10;          // mark1
        carry = Math.floor((p1.val + p2.val + carry) / 10); // mark1
        lresPointer.val = left;                             // mark2
        lresPointer = lresPointer.next;                     // mark2
        p1 = p1.next;                                       // mark3
        p2 = p2.next;                                       // mark3
    }
    if (carry == 1) {
        lresPointer.val = carry;                            // mark4
    }
    return lresHead;
};
```

这个代码有很多错误，一一来作说明吧。

- mark1  
这里没有考虑到`p1`或`p2`为`null`的情况，当该链表已被遍历完时，`p`为`null`，它是没有`val`属性的，因此需要设置默认值0。

- mark2  
这里对链表的初始化是错误的，最初的`lresPointer=lresHead=null`，是根本没有`val`属性的呀！每次新增节点不能简单粗暴而错误地设置`val`，而是要`new ListNode()`，同时还要注意第一个节点和后续节点的区别。

- mark3  
这里同样没有考虑到`p1`或`p2`为`null`的情况，当`p`为`null`时，它是没有`next`属性的，因此需要判断后再更改指向。

- mark4  
这里的修改需要和mark2处的修改相结合，要避免覆盖倒数第二个节点值的情况。

### v2: Accepted

```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function (l1, l2) {
    var p1 = l1, p2 = l2, carry = 0, lresHead = null, lresPointer = lresHead;
    while (p1 != null || p2 != null) {
        var num1 = 0, num2 = 0;
        if (p1) {
            num1 = p1.val;
        }
        if (p2) {
            num2 = p2.val;
        }
        var left = (num1 + num2 + carry) % 10;
        carry = Math.floor((num1 + num2 + carry) / 10);
        if (lresHead == null) {
            lresHead = new ListNode(left);
            lresPointer = lresHead;
        } else {
            lresPointer.next = new ListNode(left);
            lresPointer = lresPointer.next;
        }
        if (p1) {
            p1 = p1.next;
        }
        if (p2) {
            p2 = p2.next;
        }
    }
    if (carry == 1) {
        lresPointer.next = new ListNode(carry);
        lresPointer = lresPointer.next;
    }
    return lresHead;
};
```

## 反思
- 对链表的操作太不熟悉了

## 标签
- 链表
