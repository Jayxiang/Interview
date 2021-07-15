# iOS 面试-数据结构和算法

#### 常见的数据结构类型

```
1、集合结构 说白了就是一个集合，就是一个圆圈中有很多个元素，元素与元素之间没有任何关系 这个很简单
2、线性结构 说白了就是一个条线上站着很多个人。 这条线不一定是直的。也可以是弯的。也可以是直的 
相当于一条线被分成了好几段的样子 （发挥你的想象力）。 线性结构是一对一的关系
3、树形结构 说白了 做开发的肯定或多或少的知道 xml 解析 树形结构跟他非常类似。
也可以想象成一个金字塔。树形结构是一对多的关系
4、图形结构：比较复杂，无穷，无边，无向
```
#### 数据结构的存储方式

```
数据结构的存储一般常用的有两种 顺序存储结构 和 链式存储结构
```
#### 数组和链表区别

```
数组：数组元素在内存上连续存放，可以通过下标查找元素；
插入、删除需要移动大量元素，比较适用于元素很少变化的情况
链表：链表中的元素在内存中不是顺序存储的，查找慢，插入、删除只需要对元素指针重新赋值，效率高
```
#### 常用算法及其复杂度
![常见排序算法-图片来自小码哥](./image/排序.png)

#### 链表反转
```
//一个链表
struct Node {
    int data;
    struct Node *next;
};
//链表反转 1.迭代
struct Node *reverseList(struct Node *head) {
    struct Node *newH = NULL;
    while (head != NULL) {
        struct Node *temp = head->next;
        head->next = newH;
        newH = head;
        head = temp;
    }
    return newH;
}
//链表反转 2.递归
struct Node *reverseLiset2(struct Node *head) {
    if (head == NULL || head->next == NULL) {
        return head;
    }
    //注意,此处的操作是一直循环到链表末尾
    struct Node *newHead = reverseLiset2(head->next);
    head->next->next = head;
    head->next = NULL;
    printList(head);
    return newHead;
}
```
#### 什么是二叉树
```
树形结构下，两个节点以内都称之为二叉树，不存在大于 2 的节点，分为左子树右子树有顺序不能颠倒
二叉树有五种表现形式：
1.空的树(没有节点)可以理解为什么都没 像空气一样
2.只有根节点。
3.只有左子树
4.只有右子树
5.左右子树都有 
```

### 算法学习参考
[算法代码在线演示](https://algorithm-visualizer.org)  
[数据结构和算法动态可视化](https://visualgo.net/zh)
[leetcode解题之路](https://github.com/azl397985856/leetcode)