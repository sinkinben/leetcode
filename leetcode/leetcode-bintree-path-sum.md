## Path Sum

Problems:

- [112. Path Sum](https://leetcode.com/problems/path-sum/): 给定 `sum`，问是否存在 `root` 到叶子的路径，其和为 `sum` （前序遍历）。
- [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/)：给定 `sum`，返回所有 `root` 到叶子，且和为 `sum` 的路径（前序遍历）。
- [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/): 给定 `sum`，返回和为 `sum` 的路径的数量，此处的「路径」不需要从根开始，也不需要在叶子结束，但路径必须是向下延伸的（前序遍历 + 前缀和）。
- [666. Path Sum IV](https://leetcode-cn.com/problems/path-sum-iv/)：给定 `nums`，它以特殊形式存储了一棵二叉树。求所有「路径和」之和。此处的路径是从根到叶子的路径。



## Path Sum

```cpp
class Solution 
{
public:
    bool res = false;
    bool hasPathSum(TreeNode* root, int target) 
    {
        preorder(root, 0, target);
        return res;
    }
    
    void preorder(TreeNode *node, int sum, int target)
    {
        if (node == nullptr || res) return;
        
        sum += node->val;
        if (!node->left && !node->right && sum == target)
        {
            res = true;
            return;
        }
        preorder(node->left, sum, target);
        preorder(node->right, sum, target);
    }
};
```





## Path Sum II

```cpp
using vec = vector<int>;
using vec2 = vector<vec>;
class Solution {
public:
    vec2 res;
    vec path;
    vec2 pathSum(TreeNode* root, int target)
    {
        preorder(root, target, 0);
        return res;
    }

    void preorder(TreeNode *node, int target, int cur)
    {
        if (!node) return;

        cur += node->val, path.push_back(node->val);
        if (!node->left && !node->right && cur == target)
            res.emplace_back(path);

        preorder(node->left, target, cur);
        preorder(node->right, target, cur);
        cur -= node->val, path.pop_back();
    }
};
```



## Path Sum III

**前缀和**

```cpp
class Solution 
{
public:
    vector<int> prefix = {0};
    int res = 0;
    int pathSum(TreeNode* root, int target) 
    {
        preorder(root, target);
        return res;
    }
    
    void preorder(TreeNode *node, int target)
    {
        if (node == nullptr) return;
        
        prefix.push_back(prefix.back() + node->val);
        int n = prefix.size();
        for (int i = 0; i < n - 1; ++i)
            res += (prefix[n - 1] - prefix[i] == target);
        preorder(node->left, target);
        preorder(node->right, target);
        prefix.pop_back();
    }
};
```

**哈希 + 前缀和计数**

```cpp
class Solution {
public:
    unordered_map<int64_t, int> mp = {{0, 1}};
    int res = 0;
    int pathSum(TreeNode* root, int targetSum)
    {
        preorder(root, 0, targetSum);
        return res;
    }

    void preorder(TreeNode *node, int64_t sum, int target)
    {
        if (node == nullptr)
            return;
        sum += node->val;

        // suppose prefix[j] = sum - target, prefix[i] = sum, 
        // prefix[i] - prefix[j], this means we have the range whose sum is target
        if (mp.count(sum - target))
            res += mp[sum - target];
        mp[sum]++;
        preorder(node->left, sum, target);
        preorder(node->right, sum, target);
        if ((--mp[sum]) == 0)
            mp.erase(sum);
    }
};
```



## Path Sum VI

If the depth of a tree is smaller than `5`, then this tree can be represented by an array of three-digit integers. For each integer in this array:

- The hundreds digit represents the depth `d` of this node where `1 <= d <= 4`.
- The tens digit represents the position `p` of this node in the level it belongs to where `1 <= p <= 8`. The position is the same as that in a full binary tree.
- The units digit represents the value `v` of this node where `0 <= v <= 9`.
- Given an array of ascending three-digit integers nums representing a binary tree with a depth smaller than 5, return the sum of all paths from the root towards the leaves.

It is guaranteed that the given array represents a valid connected binary tree.

**Constraints:**

- `1 <= nums.length <= 15`
- `110 <= nums[i] <= 489`
- nums represents a valid binary tree with depth less than `5`.



**Example 1**

<img src="https://assets.leetcode.com/uploads/2021/04/30/pathsum4-1-tree.jpg" />

```
Input: nums = [113,215,221]
Output: 12
Explanation: The tree that the list represents is shown.
The path sum is (3 + 5) + (3 + 1) = 12.
```

**Example 2**

<img src="https://assets.leetcode.com/uploads/2021/04/30/pathsum4-2-tree.jpg" />

```
Input: nums = [113,221]
Output: 4
Explanation: The tree that the list represents is shown. 
The path sum is (3 + 1) = 4.
```

<br/>

**Solution**

- Use a `map<int, int> table` to record `position -> val`, and `position = <level, index>` is a two-digits number, where ten-digit `level` denote node's level, unit-digit `index` denote node's index in its level.
- Given a node's position `<level, index>`, we can get:
  - its left-child by `<level + 1, 2 * index - 1>` , 
  - its right-child by `<level + 1, 2 * index>` , 
  - its key in `table` is `level * 10 + index`.

Based on this definition above, we can perform a **pre-order** on the `table`-map, which stores our binary tree.

```cpp
class Solution
{
public:
    int res = 0;
    map<int, int> table;  // (level, pos) -> val
    int pathSum(vector<int>& nums) 
    {
        if (nums.empty()) return 0;
        for (int x : nums)
            table[x / 10] = x % 10;
        preorder(1, 1, 0);
        return res;
    }
    
    int getKey(int level, int pos) { return level * 10 + pos; }
    
    void preorder(int level, int pos, int path)
    {
        int key = getKey(level, pos);
        
        // nullptr node
        if (table.count(key) == 0) return;
        
        path += table[key];
        
        // leaf node
        int lkey = getKey(level + 1, 2 * pos - 1);
        int rkey = lkey + 1;
        if (table.count(lkey) == 0 && table.count(rkey) == 0)
            res += path;
        
        preorder(level + 1, 2 * pos - 1, path);
        preorder(level + 1, 2 * pos, path);
        
    }
};
```

