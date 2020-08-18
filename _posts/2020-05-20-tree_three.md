---
layout: post
title: 树（三），二叉搜索树
categories: [Python, Data Structure]
---

二叉搜索树(Binary Search Tree)，又名二叉排序树(Binary Sort Tree)

<!-- more -->
## 性质
1. 若左子树不为空，则左子树上所有节点的值均小于或等于它的根节点的值

2. 若右子树不为空，则右子树上所有节点的值均大于或等于它的根节点的值

3. 左、右子树也分别为二叉搜索树

## 代码
```python
class TreeNode:
    """二叉搜索树节点的定义"""

    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None


class OperationTree:
    """二叉搜索树操作"""
    def insert(self, root, val):
        """二叉搜索树插入操作"""
        if root is None:
            root = TreeNode(val)
        elif val < root.val:
            root.left = self.insert(root.left, val)
        elif val > root.val:
            root.right = self.insert(root.right, val)
        return root

    def query(self, root, val):
        """二叉搜索树查询操作"""
        if root is None:
            return False
        if root.val == val:
            return True
        elif val < root.val:
            return self.query(root.left, val)
        elif val > root.val:
            return self.query(root.right, val)

    def findMin(self, root):
        """查找二叉搜索树中最小值点"""
        if root.left:
            return self.findMin(root.left)
        else:
            return root

    def findMax(self, root):
        """查找二叉搜索树中最大值点"""
        if root.right:
            return self.findMax(root.right)
        else:
            return root

    def delNode(self, root, val):
        """删除二叉搜索树中值为val的点"""
        if root is None:
            print("无此节点")
            return
        if val < root.val:
            root.left = self.delNode(root.left, val)
        elif val > root.val:
            root.right = self.delNode(root.right, val)
        # 当val == root.val时，分为三种情况：只有左子树或者只有右子树、有左右子树、即无左子树又无右子树
        else:
            if root.left and root.right:
                # 既有左子树又有右子树，则需找到右子树中最小值节点
                temp = self.findMin(root.right)
                root.val = temp.val
                # 再把右子树中最小值节点删除
                root.right = self.delNode(root.right, temp.val)
            elif root.right is None and root.left is None:
                # 左右子树都为空
                root = None
            elif root.right is None:
                # 只有左子树
                root = root.left
            elif root.left is None:
                # 只有右子树
                root = root.right
        return root

    def printTree(self, root):
        # 打印二叉搜索树(中序打印，有序数列)
        if root is None:
            return
        self.printTree(root.left)
        print(root.val, end=' ')
        self.printTree(root.right)


if __name__ == '__main__':
    List = [17, 5, 35, 2, 11, 29, 38, 9, 16, 8]
    root = None
    op = OperationTree()
    for val in List:
        root = op.insert(root, val)
    print('中序打印二叉搜索树：', end=' ')
    op.printTree(root)
    print('')
    print('根节点的值为：', root.val)
    print('树中最大值为:', op.findMax(root).val)
    print('树中最小值为:', op.findMin(root).val)
    print('查询树中值为5的节点:', op.query(root, 5))
    print('查询树中值为100的节点:', op.query(root, 100))
    print('删除树中值为16的节点:', end=' ')
    root = op.delNode(root, 16)
    op.printTree(root)
    print('')
    print('删除树中值为5的节点:', end=' ')
    root = op.delNode(root, 5)
    op.printTree(root)
```