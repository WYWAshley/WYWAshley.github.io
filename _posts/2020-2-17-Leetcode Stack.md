<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {
skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
inlineMath: [['$','$']]
}
});
</script>

---
layout: post
title: Stack Summary
categories: [Leetcode, Algorithm]
description: Problems with Stack on Leetcode

keywords: Leetcode, Stack
---

力扣网站上关于 Stack 问题做一个总结




#### [173. 二叉搜索树迭代器](https://leetcode-cn.com/problems/binary-search-tree-iterator/)

实现一个二叉搜索树迭代器。你将使用二叉搜索树的根节点初始化迭代器。

调用 `next()` 将返回二叉搜索树中的下一个最小的数。

示例：

<img src="/images/posts/Leetcode Stack/1.png"/>

```
BSTIterator iterator = new BSTIterator(root);
iterator.next();    // 返回 3
iterator.next();    // 返回 7
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 9
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 15
iterator.hasNext(); // 返回 true
iterator.next();    // 返回 20
iterator.hasNext(); // 返回 false
```

提示：

- `next()` 和 `hasNext()` 操作的时间复杂度是 O(1)，并使用 O(*h*) 内存，其中 *h* 是树的高度。
- 你可以假设 `next()` 调用总是有效的，也就是说，当调用 `next()` 时，BST 中至少存在一个下一个最小的数。

**思路：**

&emsp;&emsp;① 想清楚二叉搜索树的特性，左边比节点小，右边比节点大，所以找当前最小其实就是==中序遍历==；

&emsp;&emsp;② 如何做到中序遍历呢？我们已有的条件只有 root 一个根节点，刚开始的最小值肯定是整棵树的最左边，其次是最左边的根节点，然后注意了，是最左边的右节点。这里就告诉我们，每次我们都需要记住那个根节点，否则找不到它的右节点。那么这个根节点怎么记住呢？很简单，我们在每次打印出它的值的时候不就知道它了吗，那个时候以 TreeNode 形式记住它；

&emsp;&emsp;③ 因为是先递归左子树，然后输出根节点，然后递归右子树，所以我们考虑用 ==stack 数据结构==，先进后出，==每次最后一个放进去的是目前最左边的==，然后输出，然后放它的右子树的左侧树。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class BSTIterator:

    def __init__(self, root: TreeNode):
        self.stack = []
        # 从根节点开始依次把左节点放入栈中
        while root:
            self.stack.append(root)
            root = root.left
        

    def next(self) -> int:
        """
        @return the next smallest number
        """
        tmp = self.stack.pop()
        res = tmp.val
        # 此时res跳出就应该把res的右节点从根节点开始依次向左走放入栈中
        # 注意这里的tmp已经入栈过一次了
        tmp = tmp.right
        while tmp:
            self.stack.append(tmp)
            tmp = tmp.left

        return res
        

    def hasNext(self) -> bool:
        """
        @return whether we have a next smallest number
        """
        return self.stack != []


# Your BSTIterator object will be instantiated and called as such:
# obj = BSTIterator(root)
# param_1 = obj.next()
# param_2 = obj.hasNext()
```

另外，这里附加上用 python3 实现 stack 的其他功能：

```python
### 使用list实现栈的数据接口，先进后出#####
### Stack的几大方法：push,pop,peek,find,empty,full,length,reverse,__str__
import copy
class Stack():

    def __init__(self,len): ##初始化函数，参数是指定stack的长度，用list data[]存储数据
        self.len = len
        self.data = []

    def push(self,var):   ## 1.入栈操作：判断当前栈的长度是否满，不满则直接append到data中；满了提示错误。
        if len(self.data)<self.len:
            self.data.append(var)
            print("Pushed in")
            return True
        else:
            print("Stack Full,Data not pushed in")
            return False

    def pop(self):  ##2：出站操作：直接调用list的pop（）方法
        return self.data.pop()

    def peek(self): ##3.返回栈顶元素：直接返回列表的最后一个元素
        return self.data[-1]

    def empty(self): ##4.判空：直接判断data是否为空
        return bool(self.data)

    def full(self): ##5.判满：直接对比实际长度与定义的栈长度是否一致
        return len(self.data)==self.len

    def find(self,var): ##6.查找元素的index：直接调用index方法，返回索引。
         try:
             return self.data.index(var)
         except ValueError:
             return -1
    def length(self): ## 7.长度：直接len函数返回
        return len(self.data)

    def reverse(self): ## 8.栈反转：通过copy一个新的栈，然后依次入栈并返回
        data = copy.copy(self.data)
        data.reverse()
        stack = Stack(len(data))
        for i in data:
            stack.push(i)
        return stack

    def __str__(self): ##9.定义__str__方法，print函数
        return str(self.data)

if __name__ == '__main__':
    stack = Stack(5)
    # print(stack.empty())
    stack.push(1)
    stack.push(2)
    stack.push(1)
    stack.push(3)
    stack.push(4)
    stack.push(5)
    print(stack)
    print(stack.reverse())
    print(stack)
    # print(stack.full())
    # print(stack.peek())
    # print(stack.find(1))
    # print(stack.pop())
    # print(stack.pop())
    # print(stack.peek())
```

<br/>

#### [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

给定一个二叉树，返回它的中序遍历，用迭代算法完成。

示例:

```
输入: [1,null,2,3]
   1
    \
     2
    /
   3

输出: [1,3,2]
```

**思路：**

​		①这道题和上一题一样，也是二叉树的中序遍历。中序遍历的重点就在于要先把根节点放进栈中，先输出左子树，然后栈pop，然后对根节点（即pop出来的东西）的右子树做重复的操作。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        stack = []
        ans = []
        while root:
            stack.append(root)
            root =  root.left

        while stack:
            tmp = stack.pop()
            # 哈哈其实只是把上一道题的return res换成了ans.append
            ans.append(tmp.val)
            tmp = tmp.right
            while tmp:
                stack.append(tmp)
                tmp = tmp.left

        return ans
```

<br/>

#### [103. 二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

给定一个二叉树，返回其节点值的锯齿形层次遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

例如：
给定二叉树 `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

返回锯齿形层次遍历如下：

```
[
  [3],
  [20,9],
  [15,7]
]
```

**思路：**

​		①这道题是考二叉树的层次遍历的，回忆一下层次遍历我们是用队列实现的，但是这里的层次遍历有一个之字形的风格。如果我们在放进队列的时候，一直是先进左子树再进右子树，那么输出的时候就会有一个跃迁。==如何实现之字形呢？就是我们要用栈实现先进后出。==以题目举例说明，刚开始我们append 3，然后pop出来，接着放进去3的子节点9和20，然后pop出来的时候顺序是20和9，同时在pop出来的时候也分别把他们的子节点放进去，即append 7 15 null null，那么出栈的顺序就是null null 15 7，就符合题目要求了。

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
import copy

class Solution:
    def zigzagLevelOrder(self, root: TreeNode) -> List[List[int]]:
        stack = []
        stack.append(root)
        stack2 = []
        ans = []
        tmp = []
        # 用flag标识此时应该先放左子节点还是右子节点
        flag = True
        # 因为子节点不能入同一个栈，会和父节点混乱，所以要用两个栈交替实现
        while stack or stack2:
            while stack:
                root = stack.pop()
                if root:
                    tmp.append(root.val)
                    if flag:
                        stack2.append(root.left)
                        stack2.append(root.right)
                    else:
                        stack2.append(root.right)
                        stack2.append(root.left)
            # 这里注意一下浅拷贝还是深拷贝的问题，因为list是一个可变对象
            stack = copy.deepcopy(stack2)
            stack2.clear()
            flag = not flag
            # 这里得使用tmp，因为每次放进去的是一个list，答案是个二维数组
            if tmp:
                ans.append(copy.deepcopy(tmp))
            tmp.clear()

        return ans
```

<br/>

#### [331. 验证二叉树的前序序列化](https://leetcode-cn.com/problems/verify-preorder-serialization-of-a-binary-tree/)

序列化二叉树的一种方法是使用前序遍历。当我们遇到一个非空节点时，我们可以记录下这个节点的值。如果它是一个空节点，我们可以使用一个标记值记录，例如 `#`。

```
     _9_
    /   \
   3     2
  / \   / \
 4   1  #  6
/ \ / \   / \
# # # #   # #
```

例如，上面的二叉树可以被序列化为字符串 `"9,3,4,#,#,1,#,#,2,#,6,#,#"`，其中 `#` 代表一个空节点。

给定一串以逗号分隔的序列，验证它是否是正确的二叉树的前序序列化。编写一个在不重构树的条件下的可行算法。

每个以逗号分隔的字符或为一个整数或为一个表示 `null` 指针的 `'#'` 。

你可以认为输入格式总是有效的，例如它永远不会包含两个连续的逗号，比如 `"1,,3"` 。

示例 1:

```
输入: "9,3,4,#,#,1,#,#,2,#,6,#,#"
输出: true
```

示例 2:

```
输入: "1,#"
输出: false
```

示例 3:

```
输入: "9,#,#,1"
输出: false
```

**思路：**

​		①从题意可得，只有当子节点是null才能说明这个根节点被结束，所以我们的重点是找到根节点，然后逐个地丢弃已经完成了的根节点；

​		②接着看每有一个#就说明一边的结束，但是真正能代表节点结束的是连着两个##，比如我们看题中的“2 # 6 # #”，6结束了，那么如何判断2结束了呢？就是在丢弃6的时候加一个#进去，这不就有连着的两个#了；

​		③然后我们再看，要从后往前丢弃节点，所以使用栈数据结构

​		④因为丢弃的时候我们是按照层次排列的（肯定要根据自己的儿子节点来丢弃父节点，孙子节点是不能判断的），所以我们要保证读入的时候也是层次读取的，所以我们不能一次性的读取所有数字入栈，而是边读边选择丢弃，一旦出现了栈中仅剩一个值的时候，那应该就是根节点了，而且此时根节点应该是#。所以这个可以作为判断标准

```python
class Solution:
    def isValidSerialization(self, preorder: str) -> bool:
        stack = []
        pre = preorder.split(',')
        for i in range(len(pre)):
            stack.append(pre[i])
            # Check if the last and second last is None, if so, the root canbe seen as None and change it to be None#
            while len(stack) >= 3 and stack[-1] == stack[-2] == '#':
                stack[-3] = '#'
                stack.pop()
                stack.pop()
            # if and only if when string ends stack equals to ['#']
            if len(stack) == 1 and stack[0] == '#' and i != len(pre)-1:
                return False
                
        if len(stack) == 1 and stack[0] == '#':
            return True
        else:
            return False
```

<br/>

#### [735. 行星碰撞](https://leetcode-cn.com/problems/asteroid-collision/)

给定一个整数数组 `asteroids`，表示在同一行的行星。

对于数组中的每一个元素，其绝对值表示行星的大小，正负表示行星的移动方向（正表示向右移动，负表示向左移动）。每一颗行星以相同的速度移动。

找出碰撞后剩下的所有行星。碰撞规则：两个行星相互碰撞，较小的行星会爆炸。如果两颗行星大小相同，则两颗行星都会爆炸。两颗移动方向相同的行星，永远不会发生碰撞。

示例 1:

```
输入: 
asteroids = [5, 10, -5]
输出: [5, 10]
解释: 
10 和 -5 碰撞后只剩下 10。 5 和 10 永远不会发生碰撞。
```

示例 2:

```
输入: 
asteroids = [8, -8]
输出: []
解释: 
8 和 -8 碰撞后，两者都发生爆炸。
```

示例 3:

```
输入: 
asteroids = [10, 2, -5]
输出: [10]
解释: 
2 和 -5 发生碰撞后剩下 -5。10 和 -5 发生碰撞后剩下 10。
```

示例 4:

```
输入: 
asteroids = [-2, -1, 1, 2]
输出: [-2, -1, 1, 2]
解释: 
-2 和 -1 向左移动，而 1 和 2 向右移动。
由于移动方向相同的行星不会发生碰撞，所以最终没有行星发生碰撞。
```

**思路：**

​		①这其实是比大小的另一个版本，就是新的数来了，如果他会和原来的数字相撞，那么是谁赢呢？赢了之后他会不会继续和更原先存在的数字相撞呢？所以就是这个思路联想到了使用栈，因为栈可以连续pop，就像是连续赢一样的效果

​		②只有左边的数字（栈中原有的）是往右走，右边的数字（新想要加入）往左走的时候才会有相撞的事情发生，所以在入栈前先判断是不是会相撞，再判断谁赢

​		③可以看出当新加入的球把原先的最右边的球吞了之后，他还会继续找栈中第二右边的球，进行如②一模一样的判断，所以可以写成一个循环，只有碰到终止条件（新球没了、栈被吃空了、终于碰到同方向的球了）再跳出循环

```python
class Solution:
    def asteroidCollision(self, asteroids: List[int]) -> List[int]:
        stack = []
        flag = True
        for item in asteroids:
            while True:
                # when stack in empty or we find a partner, put it in.
                if not stack or item < 0 and stack[-1] < 0 or item > 0 and stack[-1] > 0 or item > 0 and stack[-1] < 0:
                    stack.append(item)
                    break
                else:
                    # item be swallowed, no change for stack
                    if stack[-1] > -item:
                        break
                    # same weights, both died
                    elif stack[-1] == -item:
                        stack.pop()
                        break
                    # item wins
                    else:
                        stack.pop()      
                    
        return stack
```

<br/>

#### [1124. 表现良好的最长时间段](https://leetcode-cn.com/problems/longest-well-performing-interval/)

给你一份工作时间表 `hours`，上面记录着某一位员工每天的工作小时数。

我们认为当员工一天中的工作小时数大于 `8` 小时的时候，那么这一天就是「**劳累的一天**」。

所谓「表现良好的时间段」，意味在这段时间内，「劳累的天数」是严格 **大于**「不劳累的天数」。

请你返回「表现良好时间段」的最大长度。

示例 1：

```
输入：hours = [9,9,6,0,6,6,9]
输出：3
解释：最长的表现良好时间段是 [9,9,6]。
```

**思路：**

1. 观察到题目中给出数组中的元素有且只有两种,分别是大于8和小于等于8,而具体的数值没有意义.所以简单起见,我们用1代表大于8的元素，用-1代表小于等于8的元素,得到一个数组[1,1,-1,-1,-1,-1,1]记为arr。

2. 现在题目变成了,我们需要找到一个子列，子列中1的元素数量严格大于-1的元素数量。

3. 继续简化: 也就是需要找到一个子列，子列中所有元素的和大于0，这是==前缀和==的思想：

4. 这时自然可以想到直接**暴力遍历所有子列**，找到和大于0且长度最大的子列即可。

5. 这时观察到，在遍历内层循环j的过程中， 给任意一个 $i < j1 < j2$, 如果 $prefixSum[i] < prefixSum[j2]$, 那么 $(i,j1)$ 一定不会是答案。也就是说,如果 $(i, j2)$ 符合条件, 那么它们中间的所有j1都不需要再检查, 所以我们可以从右往左遍历j:

   ```python
   i = 0
   j = 0
   res = 0
   num = len(prefixSum)
   for i in range(num):
       for j in range(num - 1, i, -1):  # j现在是从7遍历到1
           if prefixSum[i] < prefixSum[j]:
               res = max(j - i, res)
               continue  # 说明此时剩下的(i,j)不是候选项
   ```

6. 又观察到, 在遍历外层循环 i 的过程中, 在对于任意的一个 $i < i1 < j$, 如果 $prefixSum[i1] >= prefixSum[i]$，那么 $(i1, j)$ 一定不会是答案。因为:

   ​	①如果 $prefixSum[j] > prefixSum[i1]$, 那么$(i1, j)$ 一定不会是答案，因为$(i, j)$更长。

   ​	②如果$ prefixSum[j] < prefixSum[i1]$, 那么 $(i1, j)$ 也一定不会是答案,因为我们要找 $prefixSum[j] -prefixSum[i1] > 0$ 的 $(i, j)$ 

   所以，这时我们需要从头遍历一遍 prefixSum, 找到一个严格单调递减的数组。比如当前这个测试用例,只有[0, 5, 6]可能是 i 的候选项。

   ```python
   stk = []
   for i in range(len(prefixSum)):
       if len(stk) == 0 or prefixSum[stk[-1]] > prefixSum[i]:
           stk.append(i)  # 因为最终需要的答案是最长距离,需要下标来计算,所以这里存储下标
       print(stk)  # [0, 5, 6]
   ```

7. 对于一个 j, 如果它满足 $prefixSum[j] > prefixSum[stk[0]]$, 那么$(0, j)$是候选项, 但是由于 stk 是单调递减的,所以 $prefixSum[j]$ 也是 $>prefixSum[stk[0 + x]]$,那么 $(stk[0 + x], j)$ 也是候选项。

   对于一个j, 如果它满足$ prefixSum[j] < prefixSum[stk[0]]$, 那么$(0, j)$不是候选项, 但是 $prefixSum[j],prefixSum[stk[0 + x]]$ 的大小关系无法判断,所以 $(stk[0 + x], j)$ 也是候选项。

   但是如果反过来, 反向遍历 stk, 对于一个 j 如果它满足 $prefixSum[j] < prefixSum[stk[-1]]$, 因为是单调递减的,所以 stk 中的其他元素都不会再小于 $prefixSum[j]$ , 所以j就可以直接被排除掉。

   再然后, 如果对于一个 j 如果它满足 $prefixSum[j] > prefixSum[stk[-1]]$, 那么 $(stk[-1], j)$ 就是候选项,此时再根据5, 对于 $stk[-1]$ 来说, j再继续向左遍历已经没有意义了,所以就可以把 $stk[-1]$ 排除掉了，而 $stk[-2]$ 及后面的元素还需要继续判断,但也不必回溯到 prefixSum 的最右端继续遍历j了。

根据以上思路, 操作刚好契合栈, 所以就用栈stk来存储[0, 5, 6], 得到最终的代码.

```python
class Solution:
    def longestWPI(self, hours: List[int]) -> int:
        # 转换成是否大于8小时的格式
        hours = [1 if h>8 else -1 for h in hours]
        start = 0
        # 累加，这样就可以用大于0表示劳累天数严格大于不劳累天数
        prefix = []
        for i in range(len(hours)):
            prefix.append(start)
            start = start + hours[i]
        prefix.append(start)

        # 找到单调递减栈
        stk = []
        for i in range(len(prefix)):
            # 当栈为空即第一天，或者栈顶大于该数，入栈
            if not stk or prefix[stk[-1]] > prefix[i]:
                stk.append(i)
        # 倒序查找在栈中的i分别对应的j
        res = 0
        end = len(prefix)-1
        for _ in range(len(stk)):
            i = stk[-1]
            for j in range(end, -1, -1):
                # 相减值就是那段时间劳累减去不劳累的天数
                if prefix[j] > prefix[i]:
                    # 这里的j-i不用加1，因为prefix开始的0是我们自己添加上去的
                    res = max(res, j-i)
                    stk.pop()
                    end = j
                    break
        return res
```

