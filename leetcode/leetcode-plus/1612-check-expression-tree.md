## Check If Two Expression Trees are Equivalent

Leetcode: [1612. Check If Two Expression Trees are Equivalent](https://leetcode-cn.com/problems/check-if-two-expression-trees-are-equivalent/)

**Description**

A binary expression tree is a kind of binary tree used to represent arithmetic expressions. Each node of a binary expression tree has either zero or two children. Leaf nodes (nodes with 0 children) correspond to operands (variables), and internal nodes (nodes with two children) correspond to the operators. In this problem, we only consider the `'+'` operator (i.e. addition).

You are given the roots of two binary expression trees, `root1` and `root2`. Return `true` if the two binary expression trees are equivalent. Otherwise, return `false`.

Two binary expression trees are equivalent if they evaluate to the same value regardless of what the variables are set to.

**Constraints**

- The number of nodes in both trees are equal, odd and, in the range `[1, 4999]`.
- `Node.val` is `'+'` or a lower-case English letter.
- It's guaranteed that the tree given is a valid binary expression tree.


Follow up: What will you change in your solution if the tree also supports the `'-'` operator (i.e. subtraction)?



**Example 1**

```text
Input: root1 = [x], root2 = [x]
Output: true
```

**Example 2**

<img src="https://assets.leetcode.com/uploads/2020/10/04/tree1.png" />

```
Input: root1 = [+,a,+,null,null,b,c], root2 = [+,+,a,b,c]
Output: true
Explaination: a + (b + c) == (b + c) + a
```

**Example 3**

<img src="https://assets.leetcode.com/uploads/2020/10/04/tree2.png" />

```
Input: root1 = [+,a,+,null,null,b,c], root2 = [+,+,a,b,d]
Output: false
Explaination: a + (b + c) != (b + d) + a
```

<br/>

**Solution**

Just compare all the leaf-nodes.

```cpp
/**
 * Definition for a binary tree node.
 * struct Node {
 *     char val;
 *     Node *left;
 *     Node *right;
 *     Node() : val(' '), left(nullptr), right(nullptr) {}
 *     Node(char x) : val(x), left(nullptr), right(nullptr) {}
 *     Node(char x, Node *left, Node *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution 
{
public:
    bool checkEquivalence(Node* root1, Node* root2) 
    {
        return getLeaves(root1) == getLeaves(root2);
    }
    
    vector<int> getLeaves(Node *node)
    {
        vector<int> leaves;
        preorder(node, leaves);
        sort(leaves.begin(), leaves.end());
        return leaves;
    }
    
    bool isLeaf(Node *node) { return !node->left && !node->right; }
    
    void preorder(Node *node, vector<int> &leaves)
    {
        if (node == nullptr) return;
        if (isLeaf(node))
            leaves.push_back(node->val);
        preorder(node->left, leaves);
        preorder(node->right, leaves);
    }
};
```

<br/>

**Follow Up 1**

If the operators set includes `'-'`, the we can push string `"-x"` (as it was a leaf node) into the vector, since `a - b = a + (-b)`.

What's more, we can use `map<string, int>` to record the leaf-nodes.

The in-order sequence of the given binary tree, is the real arithmetic expressions. Suppose the binary tree is:

```text
   -
 /   \
a     +
     / \
    b   -
       / \
      c   d
```

and its in-order sequence is `a - (b + (c - d))`. (And it is equivalent to `a + (-b) + (-c) + d` ).

For each term (operand), the number of `'-'` (from left to right) determine its sign (negative or positive). Hence the binary tree above, should return this map:

```json
{{"a", 1}, {"-b", 1}, {"-c", 1}, {"d", 1}}
```

Here is the code :-D .

```cpp
class Solution 
{
public:
    string negative = "-x", positive = "x";
    bool checkEquivalence(Node* root1, Node* root2) 
    {
        auto m1 = getLeaves(root1);
        auto m2 = getLeaves(root2);
        print(m1);
        print(m2);
        return m1 == m2;
    }

    void print(map<string, int> m)
    {
        for (auto &[k, v] : m) printf("(%s, %d) ", k.c_str(), v);
        puts("");
    }

    map<string, int> getLeaves(Node *node)
    {
        map<string, int> leaves;
        inorder(node, leaves, 0);
        return leaves;
    }
    
    bool isLeaf(Node *node) { return !node->left && !node->right; }
    
    void inorder(Node *node, map<string, int> &leaves, int minus)
    {
        if (node == nullptr) return;
        inorder(node->left, leaves, minus);
        if (isLeaf(node))
        {
            if (minus % 2 == 1) negative[1] = node->val, leaves[negative]++;
            else positive[0] = node->val, leaves[positive]++;
        }
        minus += node->val == '-';
        inorder(node->right, leaves, minus);
    }
};
```

<br/>

**Follow Up 2**

If there are 4 operators `+, -, *, /` in the expression tree, how can we solve this problem?

- Firstly, we check the variables set of these two expression (their names and number of occurrences, which can denoted by `map<string, int>`), i.e. check the leaf-nodes set. If these two variables set are not equal, return false.
  - This step cost $O(m_1 + m_2)$ time, where $m1, m2$ are the numbers of nodes of two expression trees, respectively.
- Assign a **random non-zero value** to each variable, then judge whether if values of these two expressions are equal.
  - Repeat this step for 1000 times (maybe less or more). 
  - If there exists a test that output "not equal", then return false. Otherwise, return true.
  - We can use in-order traversal to compute the value of expression tree, hence this step will cost $O(m_1 + m_2)$ time.
- Note that the **divisors in expressions must be non-zero.**

Why this dummy random-algorithm work?

- Since there are 4 operators in the expressions, the probability of mis-judgement $p$ is low, where $0 < p < 1$.
  - The mis-judgement events means that, for the given expressions `E1` and `E2`, they are not equal actually, but our algorithm return `true`.
  - If our algorithm return `false`, then it must be correct absolutely. The mis-judgements only happen when returning `true`.
- We assign random non-zero value, and this can lower the probability of mis-judge. 
  - For example, if given `E1 = a * (very complicated expr1)` and `E2 = a * (very complicated expr2)`, if we assign `a = b = 0` unfortunately, then the mis-judgement will happen.
- Suppose the probability of mis-judgement is $p = 0.9$ for each test (and 0.9 is an impossible value) . **Only if total 1000 tests were mis-judged**, our algorithm will mis-judge. Hence probability of mis-judgement of our algorithm is ${0.9} ^ {1000} \approx  10 ^ {-46}$.

