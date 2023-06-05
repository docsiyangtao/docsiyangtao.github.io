## 二分查找

二分查找算法，也称折**半搜索算法**、**对数搜索算法**，是一种在**有序数组**中查找某一特定元素的搜索算法。搜索过程**从数组的中间元素开始**，如果中间元素正好是要查找的元素，则搜索过程结束；如果某一特定元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样从中间元素开始比较。

平均复杂性：**O(log n)**

## 常用术语

1. 集合nums
2. 目标值target
3. 左下标left、右下标right，集合的查找范围

## 步骤

0. *将集合变为有序集合
1. 取left和right的中间值mid，将nums[mid]与target进行比较
2. 相等则返回对应下标
3. 不相等则将查找范围缩小至原来的一半（修改left或right）
4. 如到结束仍没有查找到target，则判定nums中没有包含target

## 模板

根据边界条件进行区分

### I、left<=right

[704. 二分查找 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-search/)

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

```java
// 查找时以左闭右闭即[left,right]为范围
class Solution {
    public int search(int[] nums, int target) {
        // 定义左边界
        int left = 0;
        // 定义右边界
        int right = nums.length - 1;
        // 这里循环条件为left<=right，因为left=right是一个合法的区间，符合[left,right]
        while (left <= right) {
            // 中间的下标
            int mid = left + (right - left) / 2;
            // 找到target，直接返回下标
            if (nums[mid] == target) {
                return mid;
            // 中间值小于target，则target在mid右侧，即(mid,right]，下次查找的范围为[mid+1,right]
            } else if (nums[mid] < target) {
                left = mid + 1;
            // 中间值大于target，则target在mid左侧，即[left,mid)，下次查找的范围为[left,mid-1]
            } else {
                right = mid - 1;
            }
        }
        // 循环到left=right还不满足，则nums[left]=nums[right]!=target，则返回-1，未找到
        return -1;
    }
}
```

### II、left<right

[35. 搜索插入位置 - 力扣（LeetCode）](https://leetcode.cn/problems/search-insert-position/)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

```java
// 由题可知结果的取值范围为[0,nums.length]
class Solution {
    public int searchInsert(int[] nums, int target) {
        int n = nums.length;
        int left = 0;
        // 取右边界为nums.length
        int right = n;
        // 数组的下标范围为[0,nums.length)，则left = nums.length时停下
        while (left < right) {
            int mid = left + (right - left) / 2;
            // 左边界最小为0
            int smallestIdx = Math.max(0, mid - 1);
            // target = nums[mid]时，结果可能为mid，因此新搜索范围右边界为原来的mid
            if (target <= nums[smallestIdx]) {
                right = mid;
            // 符合题意时直接返回
            } else if (nums[smallestIdx] < target && target <= nums[mid]) {
                return mid;
			// target > nums[mid]，此时mid一定不是结果，因此新搜索范围的left从mid+1处开始
            } else {
                left = mid + 1;
            }
        }
        // 到left=nums.length了，还没返回，则结果是nums.length
        return left;
    }
}
```

### III、left<right-1

[162. 寻找峰值 - 力扣（LeetCode）](https://leetcode.cn/problems/find-peak-element/)

峰值元素是指其值严格大于左右相邻值的元素。

给你一个整数数组 nums，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 任何一个峰值 所在位置即可。

你可以假设 nums[-1] = nums[n] = -∞ 。

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int n = nums.length;
        // 特殊情况处理
        if (n == 1) {
            return 0;
        }
        int left = 0;
        int right = n - 1;
        // 比较涉及到3个数，因此控制边界为left<right-1
        while (left < right - 1) {
            // 前 中 后 3个数
            int mid = left + (right - left) / 2;
            int midPre = mid - 1;
            int midNex = mid + 1;
            // 找出最大值，按最大值的位置情况分别进行处理
            int maxVal = Math.max(nums[midPre], Math.max(nums[mid], nums[midNex]));
            // nums[midPre]在3个数中最大，则查找前半部分，不包含mid，因为mid已经确定不是峰值了
            if (nums[midPre] == maxVal) {
                right = midPre;
            // mid位置的数最大，则mid是一个峰值，直接返回
            } else if (nums[mid] == maxVal) {
                return mid;
            // nums[midNex]在3个数中最大，则查找后半部分，不包含mid，因为mid已经确定不是峰值了
            } else {
                left = midNex;
            }
        }
        // 剩下两个值的情况，较大的为峰值
        return nums[left] > nums[right] ? left : right;
    }
}
```

## 其他题型

### I、寻找翻转点

[153. 寻找旋转排序数组中的最小值 - 力扣（LeetCode）](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)

已知一个长度为 n 的数组，预先按照升序排列，经由 1 到 n 次 旋转 后，得到输入数组。例如，原数组 nums = [0,1,2,4,5,6,7] 在变化后可能得到：
若旋转 4 次，则可以得到 [4,5,6,7,0,1,2]
若旋转 7 次，则可以得到 [0,1,2,4,5,6,7]
注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。

给你一个元素值 互不相同 的数组 nums ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 最小元素 。

```java
class Solution {
    public int findMin(int[] nums) {
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            int mid = left + (right - left) / 2;
            // 值小于右边的值，则说明右边一定是有序的，排除掉这一部分，但mid可能是最小值，因此保留
            if (nums[mid] < nums[right]) {
                right = mid;
            // 值大于右边的值，则说明左边一定是有序的，排除掉，同时mid已经不可能是最小值了，也排除
            } else {
                left = mid + 1;
            }
        }
        return nums[left];
    }
}
```

### II、二维数组搜索

[74. 搜索二维矩阵 - 力扣（LeetCode）](https://leetcode.cn/problems/search-a-2d-matrix/)

编写一个高效的算法来判断 `m x n` 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

- 每行中的整数从左到右按升序排列。
- 每行的第一个整数大于前一行的最后一个整数。

```java
// 搜索时不能从最小或最大开始搜，可以从右上角或左下角开始
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        // 矩阵的高度和宽度
        int h = matrix.length;
        int w = matrix[0].length;
        // 当前坐标（从右上角开始）
        int x = w - 1;
        int y = 0;
        // 边界条件，取坐标x y的合法范围
        while (x >= 0 && y <= h - 1) {
            int e = matrix[y][x];
            // 小于target，增大数值（坐标往下移）
            if (e < target) {
                y++;
            // 大于target，减小数值（坐标往左移）
            } else if (e > target) {
                x--;
            // 等于target，返回true
            } else {
                return true;
            }
        }
        // 搜索完了，返回false
        return false;
    }
}
```

## 总结

根据题目定义好边界，范围缩小的规则
