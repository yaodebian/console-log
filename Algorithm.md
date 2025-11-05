# Algorithm Series

## 两数之和

描述：
- 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
- 你可以假设每种输入只会对应一个答案，并且你不能使用两次相同的元素。
- 你可以按任意顺序返回答案。

示例01：
> 输入：nums = [2,7,11,15], target = 9
> 输出：[0,1]
> 解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。

示例02:
> 输入：nums = [3,2,4], target = 6
> 输出：[1,2]
> 解释：因为 nums[1] + nums[2] == 6 ，返回 [1, 2] 。

示例 3：
> 输入：nums = [3,3], target = 6
> 输出：[0,1]
> 解释：因为 nums[0] + nums[1] == 6 ，返回 [0, 1] 。

解法一：（暴力解法）
> 时间复杂度：O(n^2)
> 空间复杂度：O(1)
```js
/**
 * 解法一
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
  const len = nums.length
  for (let i = 0; i < len; i++) {
    for (let j = i + 1; j < len; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j]
      }
    }
  }
};
```

解法二：
> 时间复杂度：O(n)
> 空间复杂度：O(n)
```js
/**
 * 解法二
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
  const len = nums.length
  const map = new Map()
  for (let i = 0; i < len; i++) {
    const diff = target - nums[i]
    if (map.has(diff)) {
      return [map.get(diff), i]
    }
    map.set(nums[i], i)
  }
};
```

