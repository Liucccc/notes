# 1、链表

链表是由一组不必相连【不必相连：可以连续也可以不连续】的内存结构 【节点】，按特定的顺序链接在一起的抽象数据类型。

由于不必须按顺序存储，链表在插入的时候可以达到O(1)的，但是查找一个节点或者访问特定编号的节点则需要O(n)的时间

常用于组织**检索较少，而删除、添加、遍历较多**的数据



## 1、单链表

一个单向链表包含两个值: 当前节点的值和一个指向下一个节点的链接

![nluqM3](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/nluqM3.png)
![VGqFqd](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/VGqFqd.png)

## 2、双向链表

一个双向链表有三个整数值: 数值, 向后的节点链接, 向前的节点链接

![0ZADvz](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/0ZADvz.png)
![yQMv2T](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/yQMv2T.png)
### 3、循环链表

单向：尾节点的next指向头节点

双向：尾节点的next指向头节点，头节点的prev指向尾节点

![LhRgHz](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/LhRgHz.png)

# leetcode
## 1、[237. 删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list/)
![5Kguw6](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/5Kguw6.png)
![lB1GW9](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/lB1GW9.png)


## 2、[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)
![5PoPNX](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/5PoPNX.png)
![wtqBrf](https://gitee.com/Liuccccc/oss/raw/master/2021/06/25/wtqBrf.png)