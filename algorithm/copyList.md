#### 复制带随机指针的链表

[LeetCode第138题](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

[剑指offer第35题](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

```java
public Node copyRandomList(Node head) {
    if (head == null) {
        return null;
    }
    // 复制节点N‘到N后面，此时节点的random都为null
    Node cur = head;
    while (cur != null) {
        Node node = new Node(cur.val);
        node.next = cur.next;
        cur.next = node;
        cur = cur.next.next;
    }
    // 给复制的节点的random赋值，值为原节点的random的复制节点
    cur = head;
    while (cur != null) {
        if (cur.random != null) {
            cur.next.random = cur.random.next;
        }
        cur = cur.next.next;
    }
    // 拆分链表，还原原来的head，组装要返回的复制链表
    cur = head;
    Node resHead = head.next;
    Node res = head.next;
    while (cur != null) {
        cur.next = cur.next.next;
        cur = cur.next;
        if (res.next != null) {
            res.next = res.next.next;
            res = res.next;
        }
    }
    return resHead;
}
```
