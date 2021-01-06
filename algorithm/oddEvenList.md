#### 奇偶链表

[LeetCode第328题](https://leetcode-cn.com/problems/odd-even-linked-list/)

[LeetCode第86题](https://leetcode-cn.com/problems/partition-list/)

```java
public ListNode oddEvenList(ListNode head) {
    // 没有节点或只有一个节点的链表，直接返回
    if (head == null || head.next == null) {
        return head;
    }
    // 奇数链表哑结点
    ListNode odd = new ListNode(0);
    // 奇数链表指针
    ListNode oddCur = odd;
    // 偶数链表哑结点
    ListNode even = new ListNode(0);
    // 偶数链表指针
    ListNode evenCur = even;
    int index = 1;
    // 遍历链表
    while (head != null) {
        if (index % 2 != 0) {
            // 将节点加入奇数链表
            oddCur.next = head;
            // 奇数链表指针后移
            oddCur = oddCur.next;
        } else {
            // 将节点加入偶数链表
            evenCur.next = head;
            // 偶数链表指针后移
            evenCur = evenCur.next;
        }
        index++;
        head = head.next;
    }
    // 偶数链表尾节点后面接个null
    evenCur.next = null;
    // 将偶数链表接到奇数链表后面
    oddCur.next = even.next;
    return odd.next;
}
```
