---
title: BinaryTree基础知识

date: 2017-08-17 11:41:50

tags: [数据结构]

categories: 编程基础

---
BinaryTree基础知识
关键点：
<!-- more --> 

# 二叉树相关概念
## 二叉树（Binary Tree）
每个结点最多有两个子树的有序树，最大度数为2
```java
class Node 
{ 
    int key; 
    Node left, right; 
  
    public Node(int item) 
    { 
        key = item; 
        left = right = null; 
    } 
} 

class BinaryTree 
{ 
    // Root of Binary Tree 
    Node root; 
  
    // Constructors 
    BinaryTree(int key) 
    { 
        root = new Node(key); 
    } 
  
    BinaryTree() 
    { 
        root = null; 
    } 
}
```
## 满二叉树（Full BT）
 A Binary Tree is full if every node has 0 or 2 children；
```
       18
       /    \   
     15      20    
    /  \       
   40   50   
  /  \
 30  50
```
## 完全二叉树(Complete BT)
A Binary Tree is complete Binary Tree if all levels are completely filled except possibly the last level and the last level has all keys as left as possible.  
```
            18
       /         \  
     15           30  
    /  \         /  \
  40    50     100   40
 /  \   /
8   7  9 
```
>Practical example of Complete Binary Tree is Binary Heap.

## 完美二叉树(Perfect BT)
A Binary tree is Perfect Binary Tree in which all internal nodes have two children and all leaves are at same level.
```
           18
       /       \  
     15         30  
    /  \        /  \
  40    50    100   40
```

## 二叉搜索树(Binary Search Tree)
二叉树是一种数据结构，树有左右2个孩子，而二叉搜索树是一种特殊的二叉树
* 左孩子节点比父节点小
* 右孩子节点比父节点大

```
  2
 / \
1   3
```
