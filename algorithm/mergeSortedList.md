#### 合并有序链表

[LeetCode第21题](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

- 非递归解法

  - 时间复杂度：O(m+n)，m和n分别为两个链表的长度，每次循环只会有两个链表中的某一个元素被放入合并链表，因此最多有m+n次循环

  - 空间复杂度：O(1)

  ```java
  public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
      // 若某个链表为空，则直接返回另一个链表
  		if (l1 == null) {
  			return l2;
  		}
  		if (l2 == null) {
  			return l1;
  		}
      // 定义一个哑结点
  		ListNode dummy = new ListNode(0);
  		// 定义一个指针，该指针永远指向结果链表的尾节点
  		ListNode cur = dummy;
      // 遍历两个链表
  		while(l1 != null && l2 != null) {
        // 取较小的那个值作为新节点添加
  			if (l1.val > l2.val) {
  				cur.next = new ListNode(l2.val);
  				l2 = l2.next;
  			} else {
  				cur.next = new ListNode(l1.val);
  				l1 = l1.next;
  			}
  			cur = cur.next;
  		}
      // 剩下不为空的节点直接添加
  		cur.next = l1 != null ? l1 : l2;
  		return dummy.next;
  	}
  ```

  该实现有几行比较关键
  ```java
  ListNode cur = dummy;
  ...
  cur = cur.next
  ```

  1、开始时只有一个哑结点，cur指向的是结果链表的尾节点，只是因为结果链表只有一个节点，所以看似指向哑结点

  ![](/assets/algorithm/mergeSortedList1.jpg)

  2、当添加一个较小的节点后，cur指向了新的尾节点

  ![](/assets/algorithm/mergeSortedList2.jpg)

  3、cur不断的指向尾节点，是为了cur.next = new ListNode(...);能不断的添加节点至结果链表

  ![](/assets/algorithm/mergeSortedList3.jpg)

- 递归解法

  - 时间复杂度：O(m+n)，其中m和n分别为两个链表的长度，因为每次递归都会去掉两个链表中某个链表的头节点（直到至少有一个链表为空），函数 mergeTwoList最多把每个节点都递归到

  - 空间复杂度：O(m+n)，其中m和n分别为两个链表的长度，递归需要消耗栈空间，栈空间的大小取决于递归调用的深度，结束递归时mergeTwoLists函数最多调用m+n次

  ```java
  public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
      // 若某个链表为空，则直接返回另一个链表
      if (l1 == null) {
        return l2;
      }
      if (l2 == null) {
        return l1;
      }
      if (l1.val < l2.val) {
        // 取出较小的节点，剩下的节点认为已经通过mergeTwoLists形成一个有序的结果链表，在这里只要将结果链表接到该较小的节点后就行
          l1.next = mergeTwoLists(l1.next, l2);
          return l1;
      } else {
          l2.next = mergeTwoLists(l1, l2.next);
          return l2;
      }
  }
  ```
