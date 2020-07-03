```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
  private ListNode head;
  int[] reservor ;

  /** @param head The linked list's head.
        Note that the head is guaranteed to be not null, so it contains at least one node. */
  public Solution(ListNode head) {
    this.head=head;
    reservor = new int[1];
  }

  /** Returns a random node's value. */
  public int getRandom() {
    ListNode s = head;

    int i=0;
    // 蓄水池初始化
    while (s!=null&&i<reservor.length){
      reservor[i]=s.val;
      i++;
      s=s.next;
    }
    // 蓄水池算法
    while (s!=null){
      // 范围选择
      int d= new Random().nextInt(i+1);
      if(d<1){
        reservor[0]=s.val;
      }
      i++;
      s=s.next;
    }
    return reservor[0];
  }
}

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(head);
 * int param_1 = obj.getRandom();
 */

作者：wangziyuruc-5n
  链接：https://leetcode-cn.com/problems/linked-list-random-node/solution/382-lian-biao-sui-ji-jie-dian-xu-shui-chi-suan-fa-/
来源：力扣（LeetCode）
  著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

