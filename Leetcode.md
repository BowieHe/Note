# 递归算法

如题70，一共N级台阶，可以走1/2步，一共有多少种走法。

可以从最后往前推，从0开始，可以是走1步到达，也可以是走2步到达，因此可以递归，climb(current, target) --> `return climb(current, target) + climb(current, target)`，具体如下

```java
public int climbStairs(int n) {
  int memo[] = new int[n + 1];
  return climb(0, n, memo);
}
public int climb(int curr, int target){
  if(curr == target)return 1;
  if(curr > target)return 0;
  if(memo[curr] > 0)return memo[curr];
  memo[curr]=climb(curr+1,target,memo)+climb(curr+2,target,memo)
  return memo[curr];
}
```

思考时从最后往前思考，即current>target则不成立，返回0；如果相等则可以走到，返回1。
每个返回的1相加，就是最后的总长度，但是这样可能会超时

为了减少冗余，加入memo数列，将每步结果存在memo中，时间复杂度从O(2^n^)减少到O(n)



# 动态规划

对于可以被分解为包含最优子结构的子问题，即最优解可以从子问题最优解来构建

第i阶由以下两种方法：

1. 第(i - 1)往上爬一格
2. 第(i - 2)往上爬两格

到达第i阶的方法总和是到第(i - 1)和(i - 2)的总和
`dp[i] = dp[i - 1] + dp[i - 2]`

```java
public class Solution {
    public int climbStairs(int n) {
        if (n == 1) {
            return 1;
        }
        int[] dp = new int[n + 1];
        dp[1] = 1;
        dp[2] = 2;
        for (int i = 3; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
}
```



# 斐波那契数列

由于满足 `dp[i] = dp[i - 1] + dp[i - 2]`，可以得出dp[i] 就是第i个斐波那契数
简化上面动态规划，空间复杂度从O(n)降到O(1)

```java
public class Solution {
    public int climbStairs(int n) {
        if (n == 1) {
            return 1;
        }
        int first = 1;
        int second = 2;
        for (int i = 3; i <= n; i++) {
            int third = first + second;
            first = second;
            second = third;
        }
        return second;
    }
}
```



# 二叉树算法

设计总路线：明确一个节点要做的事情，剩下的交给框架
比如下面二叉树中所有节点值加一的代码

```java
void plusOne(TreeNode root) {
    if (root == null) return;
    root.val += 1;

    plusOne(root.left);
    plusOne(root.right);
}
```

在做二叉树问题时，设置`if (root == null) return;`不仅仅是在root = null时直接返回，也是递归到最后返回的值

# BFS（广度优先）

如果在做二叉树遍历的时候，广度优先则是新建一个ArrayList和Queue。在每一个层，使用for循环将节点加入Queue，然后在下一次循环时pop每一个节点，再将节点值放入ArrayList，循环往复，知道所有的根节点下都没有了子节点

```java
List<List<Integer>> res = new ArrayList<>();
if (root == null) return res;

LinkedList<TreeNode> queue = new LinkedList<>();
queue.offer(root);
while (!queue.isEmpty()) {
  List<Integer> vales = new ArrayList<>();
  int size = queue.size();
  for (int i = 0; i < size; i++) {
    TreeNode node = queue.pop();
    vales.add(node.val);
    if (node.left != null) queue.offer(node.left);
    if (node.right != null) queue.offer(node.right);
  }
  res.add(vales);
}
Collections.reverse(res);
return res;
```

# DFS深度优先

遍历二叉树的同时记住每一层所在的level，根据level来放入对应的值

```java
private void dfs(TreeNode root, int lelve, List<List<Integer>> list) {
  if (root == null) {//停止条件
    return ;
  }
  if (list.size() <= lelve) {
    //说明当前层，还没有开始存数据,进行初始化
    list.add(lelve, new ArrayList<Integer>());
  }
  //将当前节点的数据存储到当前层
  list.get(lelve).add(root.val);
  //继续遍历遍历下一层的数据
  dfs(root.left, lelve + 1, list);
  dfs(root.right, lelve + 1, list);
}
```



# 摩尔投票法

当需要求解众数的时候，比如一个List中出现次数大于1/2的数字，1/3数字时适用。

原理类似投票，给定两个变量，number和count。一次遍历List，如果碰到相同的count++，不同则count--。当count小于0时，number换为当前位置所在值。到最后，大于0的number就是出现次数最多的，在回去遍历数组看出现次数是否满足条件。

Tips：如果至少选出一个数值，那这个出现次数必定大于1/2 * len
至少选出两个，则这个数值必须大于1/3 * len
选择多少个，则这个数值必须大于（1/（m+1）） * len

根据选择的个数，来定义变量，如leetcode No。229

# Floyd判圈算法

如果一个链表有环，那可以考虑该算法。类似龟兔赛跑，兔子一次走两步，乌龟一次走一步。如果有结束，那兔子会先走到终点，结束循环；如果是个环，那兔子和乌龟肯定会在环中相遇。

例如Leetcode No.202

# 哨兵节点

当处理一个链表数据，需要删除其中某些项时，如果是中间节点，直接设置`preNode.next = curNode.next` 即可。但是如果删除的节点在头部就很麻烦。这个时候引入哨兵节点，作用类似伪头。参照Leetcode No.203

```java
ListNode sentine1 = new ListNode(0);
sentine1.next = head;
ListNode pre = sentine, cur = head;
```

# 小细节

如果返回值为List<List<Integer>> ans 类型：
定义时为`List<List<Integer>> ans = new ArrayList<List<Integer>>;`
增加第一个元素为：
`ans.add(new ArrayList<>());     ans.get(0).add(1);`
得到的结果是[  [ 1 ]  ]

处理String问题，比如回文字串问题时，使用正则表达式可以快速筛选，如`[^abc] `表示除abc以外任意字符；`[a-z-[bc]]`a-z不包含bc；`[a-z-[^def]]`def
通过`char[] c=s.toCharArray()`将字符串转换为char的list更好用下标处理

`>>`和`>>>`区别在于，前者是不移动符号位，而后者是符号位一起跟着移动。
如果计算一个二进制数中有多少1，可以通过`n&1`来确定最后一位是不是1，再整体数字右移。

计算质数的问题，可以创建一个大小为n的boolean数组。从2开始遍历，如果为false，则进入循环，将该下标所有倍数的数字都标为true。最后统计进入该循环的次数。

做链表循环的题目，最后返回的Node 肯定是该链表的head

小队列可以使用HashSet，来存放数据。根据`set.contains(number)`来确认是否set中存有相同的值

二叉搜索树的特征是对于所有的节点，所有左子树上的节点都小于该节点的值，而所有的右子树上节点的值都大于该值。

涉及数列中的元素不是数字就是字母的，可以建立一个长度为26的list，`int[] count = new int[26]`,通过`count[String.charAt(index) - 'a']`来在对应位置计数

使用`Arrays.copyOfRange(nums1, 0, k);`可以在不建立新数组的情况下，返回值，如Leetcode No.350

无符号二进制加减法，`a^b`可以得到低位相加之和， `a&b`可以得到进位值。重复多次，知道b为0，说明无进位，此时返回`a^b`即可

使用二分法的时候，为了应对只有一个元素的情况，设定while条件为`left <= right`较为合理，且出现两端情况时`left = mid + 1`,`right = mid -1`

涉及大写小写字母次数的问题，可以创建一个长度为128的数组来存放次数。因为ASCII一共有128个

HashSet 遵从哈希值的规律，equals是根据哈希值的计算。而TreeSet里面的数据是按顺序排列的。

ListNode解题时，如果需要返回一个新的ListNode，那么新建空的ListNode`ListNode ans = new ListNode(0)`，因为返回的需要时返回ListNode的初始值。所以需要新建另外一个`ListNode cur = ans`来对ans进行后续操作，而且能在返回的时候ans的指针指向该链表的头

