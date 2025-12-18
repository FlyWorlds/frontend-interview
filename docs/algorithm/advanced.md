# 高频算法详解

本章详细讲解前端面试中高频出现的算法题目，包括 LRU 缓存、动态规划、双指针、滑动窗口等。

## LRU 缓存

LRU (Least Recently Used) 是最常见的缓存淘汰策略，当缓存满时淘汰最久未使用的数据。

### 实现思路

使用 **哈希表 + 双向链表**：
- 哈希表：O(1) 查找
- 双向链表：O(1) 插入和删除，维护访问顺序

```
        哈希表                    双向链表
    ┌─────────────┐         ┌──────────────────────────┐
    │  key → node │         │ head ↔ node ↔ node ↔ tail │
    └─────────────┘         │  ↑                    ↑   │
                            │ 最近使用           最久未用 │
                            └──────────────────────────┘
```

### 完整实现

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();  // 哈希表：key → node

    // 虚拟头尾节点，简化边界处理
    this.head = { key: null, value: null, prev: null, next: null };
    this.tail = { key: null, value: null, prev: null, next: null };
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }

  // 获取数据
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }

    const node = this.cache.get(key);
    // 移到链表头部（标记为最近使用）
    this._moveToHead(node);
    return node.value;
  }

  // 写入数据
  put(key, value) {
    if (this.cache.has(key)) {
      // 已存在：更新值并移到头部
      const node = this.cache.get(key);
      node.value = value;
      this._moveToHead(node);
    } else {
      // 不存在：创建新节点
      const node = { key, value, prev: null, next: null };
      this.cache.set(key, node);
      this._addToHead(node);

      // 超出容量：删除尾部节点
      if (this.cache.size > this.capacity) {
        const removed = this._removeTail();
        this.cache.delete(removed.key);
      }
    }
  }

  // 添加到头部
  _addToHead(node) {
    node.prev = this.head;
    node.next = this.head.next;
    this.head.next.prev = node;
    this.head.next = node;
  }

  // 删除节点
  _removeNode(node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
  }

  // 移动到头部
  _moveToHead(node) {
    this._removeNode(node);
    this._addToHead(node);
  }

  // 删除尾部节点
  _removeTail() {
    const node = this.tail.prev;
    this._removeNode(node);
    return node;
  }
}

// 测试
const cache = new LRUCache(2);
cache.put(1, 1);       // 缓存: {1=1}
cache.put(2, 2);       // 缓存: {1=1, 2=2}
cache.get(1);          // 返回 1，缓存: {2=2, 1=1}
cache.put(3, 3);       // 淘汰 key=2，缓存: {1=1, 3=3}
cache.get(2);          // 返回 -1（未找到）
cache.put(4, 4);       // 淘汰 key=1，缓存: {3=3, 4=4}
cache.get(1);          // 返回 -1
cache.get(3);          // 返回 3
cache.get(4);          // 返回 4
```

### 使用 Map 的简化实现

```javascript
// Map 保持插入顺序，可以简化实现
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return -1;

    // 删除再添加，移到末尾（最近使用）
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    this.cache.set(key, value);

    // 超出容量，删除最早的（第一个）
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
}
```

---

## 动态规划

动态规划 (Dynamic Programming) 是将复杂问题分解为重叠子问题，通过存储子问题的解来避免重复计算。

### 解题步骤

1. **定义状态**：dp[i] 代表什么
2. **状态转移方程**：dp[i] 如何从之前的状态推导
3. **初始化**：边界条件
4. **遍历顺序**：确保计算 dp[i] 时所需的子问题已解决

### 1. 爬楼梯

```javascript
/**
 * 爬楼梯：每次可以爬 1 或 2 个台阶，求爬到第 n 阶有多少种方法
 *
 * 状态：dp[i] = 爬到第 i 阶的方法数
 * 转移：dp[i] = dp[i-1] + dp[i-2]（从 i-1 爬 1 阶，或从 i-2 爬 2 阶）
 */
function climbStairs(n) {
  if (n <= 2) return n;

  // 优化空间：只需要前两个值
  let prev1 = 1, prev2 = 2;

  for (let i = 3; i <= n; i++) {
    const curr = prev1 + prev2;
    prev1 = prev2;
    prev2 = curr;
  }

  return prev2;
}

// 示例
climbStairs(3);  // 3 (1+1+1, 1+2, 2+1)
climbStairs(5);  // 8
```

### 2. 最大子数组和

```javascript
/**
 * 给定整数数组，找出和最大的连续子数组
 *
 * 状态：dp[i] = 以 nums[i] 结尾的最大子数组和
 * 转移：dp[i] = max(nums[i], dp[i-1] + nums[i])
 */
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];

  for (let i = 1; i < nums.length; i++) {
    // 要么加入前面的子数组，要么自己单独成为新的子数组
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }

  return maxSum;
}

// 示例
maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]);  // 6 ([4, -1, 2, 1])
```

### 3. 打家劫舍

```javascript
/**
 * 相邻房屋不能同时偷，求能偷到的最大金额
 *
 * 状态：dp[i] = 偷到第 i 个房屋时的最大金额
 * 转移：dp[i] = max(dp[i-1], dp[i-2] + nums[i])
 *       （不偷第 i 个，或偷第 i 个但不能偷第 i-1 个）
 */
function rob(nums) {
  if (nums.length === 0) return 0;
  if (nums.length === 1) return nums[0];

  let prev1 = nums[0];
  let prev2 = Math.max(nums[0], nums[1]);

  for (let i = 2; i < nums.length; i++) {
    const curr = Math.max(prev2, prev1 + nums[i]);
    prev1 = prev2;
    prev2 = curr;
  }

  return prev2;
}

// 示例
rob([2, 7, 9, 3, 1]);  // 12 (2 + 9 + 1)
```

### 4. 零钱兑换

```javascript
/**
 * 给定不同面额的硬币和总金额，求凑成总金额的最少硬币数
 *
 * 状态：dp[i] = 凑成金额 i 的最少硬币数
 * 转移：dp[i] = min(dp[i], dp[i - coin] + 1)
 */
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;  // 金额 0 需要 0 个硬币

  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i && dp[i - coin] !== Infinity) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }

  return dp[amount] === Infinity ? -1 : dp[amount];
}

// 示例
coinChange([1, 2, 5], 11);  // 3 (5 + 5 + 1)
coinChange([2], 3);          // -1（无法凑成）
```

### 5. 最长递增子序列

```javascript
/**
 * 找出数组中最长严格递增子序列的长度
 *
 * 状态：dp[i] = 以 nums[i] 结尾的最长递增子序列长度
 * 转移：dp[i] = max(dp[j] + 1)，其中 j < i 且 nums[j] < nums[i]
 */
function lengthOfLIS(nums) {
  const n = nums.length;
  const dp = new Array(n).fill(1);  // 每个元素自身就是长度为 1 的子序列

  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }

  return Math.max(...dp);
}

// 二分查找优化版本 O(n log n)
function lengthOfLIS_optimized(nums) {
  const tails = [];  // tails[i] 表示长度为 i+1 的递增子序列的最小末尾值

  for (const num of nums) {
    // 二分查找第一个 >= num 的位置
    let left = 0, right = tails.length;
    while (left < right) {
      const mid = Math.floor((left + right) / 2);
      if (tails[mid] < num) {
        left = mid + 1;
      } else {
        right = mid;
      }
    }

    if (left === tails.length) {
      tails.push(num);
    } else {
      tails[left] = num;
    }
  }

  return tails.length;
}

// 示例
lengthOfLIS([10, 9, 2, 5, 3, 7, 101, 18]);  // 4 ([2, 3, 7, 101])
```

### 6. 买卖股票的最佳时机

```javascript
/**
 * 只能买卖一次，求最大利润
 */
function maxProfit(prices) {
  let minPrice = Infinity;
  let maxProfit = 0;

  for (const price of prices) {
    minPrice = Math.min(minPrice, price);
    maxProfit = Math.max(maxProfit, price - minPrice);
  }

  return maxProfit;
}

/**
 * 可以多次买卖
 */
function maxProfit2(prices) {
  let profit = 0;

  for (let i = 1; i < prices.length; i++) {
    // 只要有利润就累加
    if (prices[i] > prices[i - 1]) {
      profit += prices[i] - prices[i - 1];
    }
  }

  return profit;
}

/**
 * 最多买卖 k 次
 * dp[i][j][0/1] = 第 i 天，已交易 j 次，不持有/持有股票 的最大利润
 */
function maxProfit3(k, prices) {
  const n = prices.length;
  if (n === 0 || k === 0) return 0;

  // dp[j][0] = 交易 j 次后不持有股票的最大利润
  // dp[j][1] = 交易 j 次后持有股票的最大利润
  const dp = Array.from({ length: k + 1 }, () => [0, -Infinity]);

  for (const price of prices) {
    for (let j = 1; j <= k; j++) {
      // 卖出：之前持有，现在不持有
      dp[j][0] = Math.max(dp[j][0], dp[j][1] + price);
      // 买入：之前不持有，现在持有（买入算一次交易）
      dp[j][1] = Math.max(dp[j][1], dp[j - 1][0] - price);
    }
  }

  return dp[k][0];
}
```

---

## 双指针

双指针是使用两个指针遍历数组/字符串的技巧，常用于有序数组、链表等场景。

### 1. 两数之和（有序数组）

```javascript
function twoSum(numbers, target) {
  let left = 0, right = numbers.length - 1;

  while (left < right) {
    const sum = numbers[left] + numbers[right];

    if (sum === target) {
      return [left + 1, right + 1];  // 返回 1-indexed
    } else if (sum < target) {
      left++;  // 和太小，左指针右移
    } else {
      right--;  // 和太大，右指针左移
    }
  }

  return [-1, -1];
}

// 示例
twoSum([2, 7, 11, 15], 9);  // [1, 2]
```

### 2. 三数之和

```javascript
function threeSum(nums) {
  const result = [];
  nums.sort((a, b) => a - b);

  for (let i = 0; i < nums.length - 2; i++) {
    // 跳过重复
    if (i > 0 && nums[i] === nums[i - 1]) continue;

    let left = i + 1, right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];

      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        // 跳过重复
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;
        left++;
        right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }

  return result;
}

// 示例
threeSum([-1, 0, 1, 2, -1, -4]);  // [[-1, -1, 2], [-1, 0, 1]]
```

### 3. 盛最多水的容器

```javascript
function maxArea(height) {
  let left = 0, right = height.length - 1;
  let maxWater = 0;

  while (left < right) {
    const width = right - left;
    const h = Math.min(height[left], height[right]);
    maxWater = Math.max(maxWater, width * h);

    // 移动较短的那个边
    if (height[left] < height[right]) {
      left++;
    } else {
      right--;
    }
  }

  return maxWater;
}

// 示例
maxArea([1, 8, 6, 2, 5, 4, 8, 3, 7]);  // 49
```

### 4. 移动零

```javascript
function moveZeroes(nums) {
  let nonZeroIndex = 0;  // 指向下一个非零元素应该放的位置

  // 将所有非零元素移到前面
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== 0) {
      [nums[nonZeroIndex], nums[i]] = [nums[i], nums[nonZeroIndex]];
      nonZeroIndex++;
    }
  }
}

// 示例
const arr = [0, 1, 0, 3, 12];
moveZeroes(arr);  // [1, 3, 12, 0, 0]
```

### 5. 链表中点

```javascript
function middleNode(head) {
  let slow = head, fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }

  return slow;  // slow 指向中点（偶数个节点时指向后半部分第一个）
}
```

### 6. 判断链表是否有环

```javascript
function hasCycle(head) {
  let slow = head, fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;

    if (slow === fast) {
      return true;  // 相遇，有环
    }
  }

  return false;
}

// 找到环的入口
function detectCycle(head) {
  let slow = head, fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;

    if (slow === fast) {
      // 相遇后，一个指针回到起点，同速前进
      let ptr = head;
      while (ptr !== slow) {
        ptr = ptr.next;
        slow = slow.next;
      }
      return ptr;  // 相遇点就是环入口
    }
  }

  return null;
}
```

---

## 滑动窗口

滑动窗口是双指针的一种，用于处理连续子数组/子串问题。

### 模板

```javascript
function slidingWindow(s) {
  const window = {};  // 窗口内的统计
  let left = 0, right = 0;
  let result = 0;

  while (right < s.length) {
    // 扩大窗口
    const c = s[right];
    right++;
    // 更新窗口内数据
    window[c] = (window[c] || 0) + 1;

    // 判断是否需要收缩窗口
    while (/* 需要收缩的条件 */) {
      // 缩小窗口
      const d = s[left];
      left++;
      // 更新窗口内数据
      window[d]--;
    }

    // 更新结果
    result = Math.max(result, right - left);
  }

  return result;
}
```

### 1. 无重复字符的最长子串

```javascript
function lengthOfLongestSubstring(s) {
  const window = new Set();
  let left = 0, maxLen = 0;

  for (let right = 0; right < s.length; right++) {
    // 如果有重复，收缩窗口
    while (window.has(s[right])) {
      window.delete(s[left]);
      left++;
    }

    window.add(s[right]);
    maxLen = Math.max(maxLen, right - left + 1);
  }

  return maxLen;
}

// 示例
lengthOfLongestSubstring('abcabcbb');  // 3 ("abc")
lengthOfLongestSubstring('pwwkew');    // 3 ("wke")
```

### 2. 最小覆盖子串

```javascript
function minWindow(s, t) {
  const need = {};  // t 中字符的需求
  const window = {};  // 窗口内字符的统计

  for (const c of t) {
    need[c] = (need[c] || 0) + 1;
  }

  let left = 0, right = 0;
  let valid = 0;  // 满足需求的字符数
  let start = 0, len = Infinity;

  while (right < s.length) {
    const c = s[right];
    right++;

    if (need[c]) {
      window[c] = (window[c] || 0) + 1;
      if (window[c] === need[c]) {
        valid++;
      }
    }

    // 满足条件时收缩窗口
    while (valid === Object.keys(need).length) {
      // 更新最小覆盖子串
      if (right - left < len) {
        start = left;
        len = right - left;
      }

      const d = s[left];
      left++;

      if (need[d]) {
        if (window[d] === need[d]) {
          valid--;
        }
        window[d]--;
      }
    }
  }

  return len === Infinity ? '' : s.substring(start, start + len);
}

// 示例
minWindow('ADOBECODEBANC', 'ABC');  // "BANC"
```

### 3. 找到字符串中所有字母异位词

```javascript
function findAnagrams(s, p) {
  const result = [];
  const need = {};
  const window = {};

  for (const c of p) {
    need[c] = (need[c] || 0) + 1;
  }

  let left = 0, valid = 0;

  for (let right = 0; right < s.length; right++) {
    const c = s[right];

    if (need[c]) {
      window[c] = (window[c] || 0) + 1;
      if (window[c] === need[c]) valid++;
    }

    // 窗口大小等于 p 的长度时
    if (right - left + 1 === p.length) {
      if (valid === Object.keys(need).length) {
        result.push(left);
      }

      const d = s[left];
      if (need[d]) {
        if (window[d] === need[d]) valid--;
        window[d]--;
      }
      left++;
    }
  }

  return result;
}

// 示例
findAnagrams('cbaebabacd', 'abc');  // [0, 6]
```

---

## 二分查找

二分查找用于有序数组中查找目标值，时间复杂度 O(log n)。

### 基础模板

```javascript
// 查找目标值
function binarySearch(nums, target) {
  let left = 0, right = nums.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] === target) {
      return mid;
    } else if (nums[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }

  return -1;
}

// 查找左边界（第一个 >= target 的位置）
function lowerBound(nums, target) {
  let left = 0, right = nums.length;

  while (left < right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] < target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }

  return left;
}

// 查找右边界（最后一个 <= target 的位置）
function upperBound(nums, target) {
  let left = 0, right = nums.length;

  while (left < right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] <= target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }

  return left - 1;
}
```

### 搜索旋转排序数组

```javascript
function search(nums, target) {
  let left = 0, right = nums.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] === target) return mid;

    // 判断哪一半是有序的
    if (nums[left] <= nums[mid]) {
      // 左半部分有序
      if (nums[left] <= target && target < nums[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    } else {
      // 右半部分有序
      if (nums[mid] < target && target <= nums[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }

  return -1;
}

// 示例
search([4, 5, 6, 7, 0, 1, 2], 0);  // 4
```

---

## 链表操作

### 反转链表

```javascript
// 迭代
function reverseList(head) {
  let prev = null, curr = head;

  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }

  return prev;
}

// 递归
function reverseListRecursive(head) {
  if (!head || !head.next) return head;

  const newHead = reverseListRecursive(head.next);
  head.next.next = head;
  head.next = null;

  return newHead;
}
```

### 合并两个有序链表

```javascript
function mergeTwoLists(l1, l2) {
  const dummy = { next: null };
  let curr = dummy;

  while (l1 && l2) {
    if (l1.val <= l2.val) {
      curr.next = l1;
      l1 = l1.next;
    } else {
      curr.next = l2;
      l2 = l2.next;
    }
    curr = curr.next;
  }

  curr.next = l1 || l2;

  return dummy.next;
}
```

### K 个一组翻转链表

```javascript
function reverseKGroup(head, k) {
  const dummy = { next: head };
  let prev = dummy;

  while (true) {
    // 检查剩余节点是否足够 k 个
    let kth = prev;
    for (let i = 0; i < k; i++) {
      kth = kth.next;
      if (!kth) return dummy.next;
    }

    // 反转 k 个节点
    let curr = prev.next;
    let next = curr.next;

    for (let i = 1; i < k; i++) {
      curr.next = next.next;
      next.next = prev.next;
      prev.next = next;
      next = curr.next;
    }

    prev = curr;
  }
}
```

---

## 树的遍历

### 递归遍历

```javascript
// 前序：根 → 左 → 右
function preorder(root, result = []) {
  if (!root) return result;
  result.push(root.val);
  preorder(root.left, result);
  preorder(root.right, result);
  return result;
}

// 中序：左 → 根 → 右
function inorder(root, result = []) {
  if (!root) return result;
  inorder(root.left, result);
  result.push(root.val);
  inorder(root.right, result);
  return result;
}

// 后序：左 → 右 → 根
function postorder(root, result = []) {
  if (!root) return result;
  postorder(root.left, result);
  postorder(root.right, result);
  result.push(root.val);
  return result;
}
```

### 迭代遍历

```javascript
// 前序（栈）
function preorderIterative(root) {
  if (!root) return [];
  const result = [];
  const stack = [root];

  while (stack.length) {
    const node = stack.pop();
    result.push(node.val);
    // 先右后左入栈，这样左先出栈
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }

  return result;
}

// 中序（栈）
function inorderIterative(root) {
  const result = [];
  const stack = [];
  let curr = root;

  while (curr || stack.length) {
    // 一直向左走
    while (curr) {
      stack.push(curr);
      curr = curr.left;
    }

    curr = stack.pop();
    result.push(curr.val);
    curr = curr.right;
  }

  return result;
}

// 层序（队列）
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length) {
    const levelSize = queue.length;
    const level = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }

    result.push(level);
  }

  return result;
}
```

### 最大深度

```javascript
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

### 判断对称二叉树

```javascript
function isSymmetric(root) {
  if (!root) return true;

  function isMirror(left, right) {
    if (!left && !right) return true;
    if (!left || !right) return false;

    return left.val === right.val &&
           isMirror(left.left, right.right) &&
           isMirror(left.right, right.left);
  }

  return isMirror(root.left, root.right);
}
```

---

## 排序算法

### 快速排序

```javascript
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (left >= right) return arr;

  const pivotIndex = partition(arr, left, right);
  quickSort(arr, left, pivotIndex - 1);
  quickSort(arr, pivotIndex + 1, right);

  return arr;
}

function partition(arr, left, right) {
  const pivot = arr[right];
  let i = left;

  for (let j = left; j < right; j++) {
    if (arr[j] < pivot) {
      [arr[i], arr[j]] = [arr[j], arr[i]];
      i++;
    }
  }

  [arr[i], arr[right]] = [arr[right], arr[i]];
  return i;
}
```

### 归并排序

```javascript
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }

  return result.concat(left.slice(i)).concat(right.slice(j));
}
```

### 排序算法复杂度

| 算法 | 平均时间 | 最坏时间 | 空间 | 稳定性 |
|------|---------|---------|------|--------|
| 冒泡排序 | O(n²) | O(n²) | O(1) | 稳定 |
| 选择排序 | O(n²) | O(n²) | O(1) | 不稳定 |
| 插入排序 | O(n²) | O(n²) | O(1) | 稳定 |
| 快速排序 | O(n log n) | O(n²) | O(log n) | 不稳定 |
| 归并排序 | O(n log n) | O(n log n) | O(n) | 稳定 |
| 堆排序 | O(n log n) | O(n log n) | O(1) | 不稳定 |
