# Two Sum
## 题目描述
Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.
You may assume that each input would have **exactly** one solution, and you may not use the same element twice.

## 输入输出示例

> Given nums = [2, 7, 11, 15], target = 9,
> Because nums[0] + nums[1] = 2 + 7 = 9,
> return [0, 1].

## 解题思路
### v1: Wrong Answer
首先想到的是对数组进行排序，用左右两个指针，分别向右向左寻找。

- `left + right == target`，直接return
- `left + right < target`，说明需要让左值更大些，左指针++
- `left + right > target`，说明需要让右值更小些，右指针--

写出初步代码:

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function (nums, target) {
    var left = 0;
    var right = nums.length - 1;
    nums.sort((a, b) => {
        return a - b;
    });
    while (left < right) {
        if (nums[left] + nums[right] == target) {
            return [left, right];
        } else if (nums[left] + nums[right] < target) {
            left++;
        } else {
            right--;
        }
    }
};
```

毫无疑问这个是错的...这里的`left`和`right`是排序之后的下标，而我们需要获取的是原始下标。

### v2: Wrong Answer
意识到这个问题，那就需要在排序之前先记录好`数值:下标`对，在符合条件的情况下返回两个值对应的下标。键值对则可以直接用空对象存储。

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function (nums, target) {
    var left = 0;
    var right = nums.length - 1;
    var map = {};                  // added
    nums.forEach((num, index) => { // added
        map[num] = index;
    });
    nums.sort((a, b) => {
        return a - b;
    });
    while (left < right) {
        if (nums[left] + nums[right] == target) {
            return [map[nums[left]], map[nums[right]]]; // modified
        } else if (nums[left] + nums[right] < target) {
            left++;
        } else {
            right--;
        }
    }
};
```

在进行了非常不充分的测试之后提交，发现有错误样例：

> Input:    [3,3]
>           6
> Output:   [1, 1]
> Expected: [0,1]

说明没有考虑到数值重复的情况，因此需要在map部分进行修改，记录重复数据的所有下标值。

### v3: Accepted
由于可能有多个下标，因此想到用数组存储这些值。

- 当`map[cur]`为空，说明此时暂时只有一个下标，初始化键值对
- 当`map[cur]`非空，说明此时已有下标，将新下标push进去

考虑到题目说“每个输入仅有一个答案”，也就是即使需要的数值有重复，那也最多会重复两次(否则会有不止一个答案)，所以可以放心地使用`arr[0]`和`arr[1]`。

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function (nums, target) {
    var left = 0;
    var right = nums.length - 1;
    var map = {};
    nums.forEach((num, index) => { // modified 
        if (map[num] == undefined) {
            map[num] = [index];
        } else {
            map[num].push(index);
        }
    });
    nums.sort((a, b) => {
        return a - b;
    });
    while (left < right) {
        if (nums[left] + nums[right] == target) {
            if (nums[left] == nums[right]) {
                return [map[nums[left]][0], map[nums[right]][1]]; // modified
            } else {
                return [map[nums[left]][0], map[nums[right]][0]]; // modified
            }
        } else if (nums[left] + nums[right] < target) {
            left++;
        } else {
            right--;
        }
    }
};
```
成功通过！不过系统告诉我，时空消耗都不算特别好，因此查看题解寻找优化。

### v4: Accepted
上面的代码中，有两次循环：一次是用map来存储`数值:下标`对、一次是左右指针遍历，还有一次排序。但其实可以将两次减少为一次，也不再需要排序。
对于两次循环，优化方法是边遍历边更新map；对于一次排序，优化方法是不依靠数值大小来搜索，思路在于，我们不需要一个个遍历地去找互补项，而是可以直接访问map，在map中看是否有自己的互补项，这样就可以将复杂度从O(n)降为O(1)了。

- 当`map[cur]`为空，说明还没找到自己的互补项，那就把自己加入map
- 当`map[cur]`非空，说明找到了自己的互补项，直接return

而这种办法同样能解决数值重复的问题，先看代码：

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function (nums, target) {
    var map = {};
    for (var i = 0; i < nums.length; i++) {
        if (map[target - nums[i]] == undefined) {
            map[nums[i]] = i;
        } else {
            return [map[target - nums[i]], i];
        }
    }
};
```

举个例子：

> Input:  [2,3,4]
>         6
> Output: [0, 2]

第一次，`map[6-2]`为空，map中添加项，map为`{4: 0}`；
第二次，`map[6-3]`为空，map中添加项，map为`{4: 0, 3: 1}`；
第三次，`map[6-4]`非空，说明有4的互补项，把该值取出，与当前下标一同输出即可。

> Input:    [3,3]
>           6
> Output:   [0, 1]

第一次，`map[6-3]`为空，map中添加项，map为`{3: 0}`；
第二次，`map[6-3]`非空，说明有3的互补项，把该值取出，与当前下标一同输出即可。

果然，less is more!

## 反思
- 不要一看到题目就想当然
- 写函数别忘了return
- 测试要充分

## 标签
- 哈希表