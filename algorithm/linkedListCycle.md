#### 寻找环形链表的环起始点

[LeetCode第142题](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

- 思路

  ![](/assets/algorithm/linkedListCycle.png)

  - 快慢指针，快指针每次走2步，慢指针每次走1步

  - 相遇时，慢指针走了x+y步，快指针走了x+y+n*(y+z)步，为什么会有n，因为如果x很长的话，相遇前，快指针已经在环里面绕了n圈了

  - 因为快指针走的步数一定是慢指针的两倍，所以2*(x+y) = x+y+n*(y+z),所以x=n*(y+z)-y

  - 因此当两个指针相遇时，让慢指针回到head，快指针改为每次只走1步，慢指针走x步，快指针走n*(y+z)-y步后就会正好在环起始点再次相遇

- 代码

  ```java
  public ListNode detectCycle(ListNode head) {
      if (head == null || head.next == null) {
          return null;
      }
      ListNode slow = head;
      ListNode fast = head;
      while (fast != null && fast.next != null) {
          slow = slow.next;
          fast = fast.next.next;
          // 两指针相遇
          if (fast == slow) {
              slow = head;
              while (slow != fast) {
                  slow = slow.next;
                  fast = fast.next;
              }
              return slow;
          }
      }
      return null;
  }
  ```
