## 二叉树的遍历

遍历方法记忆tip：根据`根节点`的访问时机决定，根节点先访问就是前序，根节点在中间访问就是中序，根节点最后访问就是后序。

### 一、前序遍历（Preorder Traversal）

> 访问顺序：根节点 -> 前序遍历左子树 -> 前序遍历右子树

![image-20210705105243602](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705105243602.png)

#### 实现方式

- 递归

```java
// 递归实现前序遍历
private void preorderTraversal(Node<E> node) {
    // 退出条件
    if (node == null) return;
    // 打印节点值	
    System.out.println(node.element);
    // 前序遍历左子树
    preorderTraversal(node.left);
    // 前序遍历右子树
    preorderTraversal(node.right);
}
```

- 非递归

![image-20210705110950883](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705110950883.png)

### 二、中序遍历（Inorder Traversal）

> 访问顺序：中序遍历左子树 -> 根节点 -> 中序遍历右子树

使用场景：当需要按`升序`或`降序`获取节点时，可以使用`中序遍历`

![image-20210705111114458](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705111114458.png)

`二叉搜索树`的`中序遍历`结果是**升序（左->根->右）**或者**降序（右->根->左）**的

#### 实现方式

- 递归

```java
// 递归实现中序遍历
private void inorderTraversal(Node<E> node) {
    // 退出条件
    if (node == null) return;
    // 中序遍历左子树	
    inorderTraversal(node.left);
    // 打印节点值
    System.out.println(node.element);
    // 中序遍历右子树
    inorderTraversal(node.right);
}
```

- 非递归

![image-20210705111515896](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705111515896.png)

### 三、后序遍历（Postorder Traversal）

> 访问顺序：后序遍历左子树 -> 后序遍历右子树 -> 根节点

使用场景：当需要`先子后父`操作时，可以使用`后序遍历`

![image-20210705111704360](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705111704360.png)

#### 实现方式

- 递归

```java
// 递归实现后序遍历
private void postorderTraversal(Node<E> node) {
    // 退出条件
    if (node == null) return;
    // 后序遍历左子树
    postorderTraversal(node.left);
    // 后序遍历右子树
    postorderTraversal(node.right);
    // 打印节点值
    System.out.println(node.element);
}
```

- 非递归

![image-20210705111937944](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705111937944.png)

### 四、层序遍历（Level Order Traversal）

> 访问顺序：从上到下、从左到右依次访问每一个节点

使用场景：需要`计算二叉树高度`或`判断是否为完全二叉树`时，可使用`层序遍历`

![image-20210705112212598](https://gitee.com/Liuccccc/oss/raw/master/2021/07/05/image-20210705112212598.png)

实现方式

```java
public void levelOrderTranversal() {
    // 如果根节点为空，直接返回
    if (root == null) return;
    // 创建存储节点的队列
    Queue<Node<E>> queue = new LinkedList<>();
    //将头节点入队列
    queue.offer(root);
    //退出条件，当队列为空
    while (!queue.isEmpty()) {
        //取出队列头元素
        Node<E> node = queue.poll();
        //打印头元素
        System.out.println(node.element);
        //如果头元素左子树不为空
        if (node.left != null) {
            //将头元素左子树入队
            queue.offer(node.left);
        }
        //如果头元素右子树不为空
        if (node.right != null) {
            //将头元素右子树入队
            queue.offer(node.right);
        }
    }
}
```


