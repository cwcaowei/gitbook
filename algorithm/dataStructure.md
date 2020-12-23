#### 散列表（哈希表）

一种Key-Value映射形式的存储结构，通过Key查找Value的时间复杂度接近于O
(1)，因为其本质上是一个数组，通过哈希函数将Key转换成数组下标，不同的Key转换成的下标有可能是相同的，这种情况叫做哈希冲突，通过有如下两种方式解决：

- 开放寻址法

  当冲突时，寻找一个其他的位置，若该位置还没有元素，则使用该位置，否则继续寻找，ThreaLocal就是这样

- 链表法

  数组的每一个元素都是一个链表的头节点，当冲突时，将新的Value插入到链表尾部，JDK1.7的HashMap就是这样

#### 链表

- 单向链表

  每个节点包含存储的数据和指向下一个节点的指针next，第一个叫头节点，最后一个叫尾节点，尾节点的next指向null

- 双向链表

  每个节点包含存储的数据和指向下一个节点的指针next，同时还有个指向前置节点的prev指针，除了尾节点的next指向null，头节点的prev也指向null

#### 树

- 基本术语

  - 祖先节点

    沿树根到某一节点路径上的所有节点都是这个节点的祖先节点

  - 子孙节点

    某个节点子树中的所有节点都是这个节点的子孙节点

  - 子节点

    又称孩子节点

  - 子树

    节点下面一个子节点即为一个子树

  - 节点的度

    节点的子树个数

  - 树的度

    树的所有节点中最大的度

  - 树的深度

    根在1层，其他节点的层数是父节点的层数+1，最大的层数即为树的深度

  - 边

    除了根节点，每个节点都有一条通往上个节点的边，所以N个节点的树有N-1条边

- 二叉树

  一种每个节点最多有2个孩子节点的树，一个叫左孩子，一个叫右孩子，有如下几个性质：

  - 第k层的最大节点数为2的k-1次方

  - 深度为k的树的最大节点数为2的k次方-1

  - 若n0表示叶子节点的个数，n1表示度为1的节点的个数，n2表示度为2的节点的个数，则n0=n2+1

- 满二叉树

  所有非叶子节点都有左右两个孩子节点，且所有叶子节点都在同一层

- 完全二叉树

  对一个有n个节点的二叉树，按层级及从左到右的顺序编号，如果这个树的所有节点和同样深度的满二叉树的编号的节点位置相同，即为完全二叉树，如下图，从1到12节点，完全二叉树的节点位置和满二叉树一致，类似一种子集的概念

  ![](/assets/algorithm/fullBinaryTree.png)
  ![](/assets/algorithm/completeBinaryTree.png)

- 二叉查找树

  又称二叉排序树，主要有如下几个性质：

  - 若左子树不为空，则左子树上所有节点的值均小于根节点的值

  - 若右子树不为空，则右子树上所有节点的值均大于根节点的值

  - 左、右子树也都是二叉查找树

  - 搜索节点的时间复杂度为O(logn)

  ![](/assets/algorithm/binarySearchTree.png)

- 二叉堆

  本质上是一个完全二叉树，根节点称为堆顶，分为两种类型

  - 最大堆

    任何一个父节点的值都大于等于它的左右孩子节点的值

  - 最小堆

    任何一个父节点的值都小于等于它的左右孩子节点的值

#### 二叉树的遍历

- 深度优先遍历

  - 前序遍历

    顺序为根节点，左子树，右子树

    ![](/assets/algorithm/DLR.png)

    ```java
    public static void DLR(TreeNode root) {
        System.out.println(root);
        DLR(root.getLeftChild());
        DLR(root.getRightChild());
    }
    ```

  - 中序遍历

    顺序为左子树，根节点，右子树

    ![](/assets/algorithm/LDR.png)

    ```java
    public static void DLR(TreeNode root) {
        DLR(root.getLeftChild());
        System.out.println(root);
        DLR(root.getRightChild());
    }
    ```

  - 后序遍历

    顺序为左子树，右子树，根节点

    ![](/assets/algorithm/LRD.png)

    ```java
    public static void DLR(TreeNode root) {
        DLR(root.getLeftChild());
        DLR(root.getRightChild());
        System.out.println(root);
    }
    ```


- 广度优先遍历

  - 层序遍历

    顺序为从根节点到叶子节点，从左至右

    ![](/assets/algorithm/levelTraversal.png)

    ```java
    public static void levelTraversal(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        // 根节点入队
        queue.offer(root);
        while (!queue.isEmpty()) {
            // 队首出队
            TreeNode node = queue.poll();
            System.out.println(node);
            if (node.getLeftChild() != null) {
                // 左孩子入队
                queue.offer(node.getLeftChild());
            }
            if (node.getRightChild() != null) {
                // 右孩子入队
                queue.offer(node.getRightChild());
            }
        }
    }
    ```

#### 优先队列
