### 二叉搜索的前驱和后继

### 一、前驱节点（predecessor）

> 定义：中序遍历的前一个节点

如果是二叉搜索树，前驱节点就是前一个比他小的节点

![image-20210705113510502](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705113510502.png)

具体情况：

1. 左子树不为空

   - 比如：6，13，8
   - 前驱节点 = node.left.right.right.right....
   - 终止条件：right = null
2. 左子树为空，父节点不为空
   - 比如：5，7，9，1
   - 前驱节点 = node.parent.parent.parent....
   - 终止条件：node在parent的右子树
3. 左子树为空，父节点也为空
   - 没有前驱节点
   - 比如：没有左子树的根节点

```java
protected Node<E> predecessor(Node<E> node) {
    if (node == null) return null;
    // 前驱节点在左子树当中（left.right.right.right....）
    Node<E> p = node.left;
    if (p != null) {
        while (p.right != null) {
            p = p.right;
        }
        return p;
    }
    // 从父节点、祖父节点中寻找前驱节点
    while (node.parent != null && node == node.parent.left) {
        node = node.parent;
    }
    // node.parent == null
    // node == node.parent.right
    return node.parent;
}
```



### 二、后继节点（successor）

> 定义：中序遍历的后一个节点

如果是二叉搜索树，后继节点就是后一个比他大的节点

![image-20210705114857886](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705114857886.png)

具体情况：

1. 右子树不为空
   - 比如：1，8，10
   - 后继节点 = node.right.left.left.left....
   - 终止条件：left = null
2. 右子树为空，父节点不为空
   - 比如：11，7，3
   - 后继节点 = node.parent.parent....
   - 终止条件：node在parent的左子树
3. 右子树为空，父节点也为空
   - 那就是没有后继节点
   - 比如：没有右子树的根节点

```java
protected Node<E> successor(Node<E> node) {
    if (node == null) return null;
    // 前驱节点在左子树当中（right.left.left.left....）
    Node<E> p = node.right;
    if (p != null) {
        while (p.left != null) {
            p = p.left;
        }
        return p;
    }
    // 从父节点、祖父节点中寻找前驱节点
    while (node.parent != null && node == node.parent.right) {
        node = node.parent;
    }
    return node.parent;
}
```


