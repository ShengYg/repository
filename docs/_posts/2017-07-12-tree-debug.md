---
layout: post
title:  "tree debug"
date:   2017-07-12 14:00:00 +0800
categories: [algorithms]
tags: [tree, python, java]
description: 
---

~~~python
class TreeNode(object):
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None

def listToTreeNode(inputValues):
    root = TreeNode(inputValues[0])
    nodeQueue = [root]
    front = 0
    index = 1
    while index < len(inputValues):
        node = nodeQueue[front]
        front = front + 1

        item = inputValues[index]
        index = index + 1
        if item != None:
            leftNumber = item
            node.left = TreeNode(leftNumber)
            nodeQueue.append(node.left)

        if index >= len(inputValues):
            break

        item = inputValues[index]
        index = index + 1
        if item != None:
            rightNumber = item
            node.right = TreeNode(rightNumber)
            nodeQueue.append(node.right)
    return root

def printTree(root):
    if not root:
        return []
    queue = []
    queue.append(root)
    ret = []
    while len(queue):
        for i in range(len(queue)):
            res = []
            curNode = queue.pop(0)
            res.append(curNode.val)
            if curNode.left:
                queue.append(curNode.left)
                res.append(curNode.left.val)
            else:
                res.append('None')
            if curNode.right:
                queue.append(curNode.right)
                res.append(curNode.right.val)
            else:
                res.append('None')
            ret.append(res)
    for item in ret:
        print '{} -> {}, {}'.format(*item)

if __name__ == "__main__":
    a = [1, 2, 3, 4, None, 6, None]
    root = listToTreeNode(a)
    printTree(root)
~~~


~~~java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}

public class Solution {
    public static TreeNode listToTreeNode(String input) {
        String[] parts = input.split(",");
        String item = parts[0];
        TreeNode root = new TreeNode(Integer.parseInt(item));
        Queue<TreeNode> nodeQueue = new LinkedList<>();
        nodeQueue.add(root);

        int index = 1;
        while(!nodeQueue.isEmpty()) {
            TreeNode node = nodeQueue.remove();

            if (index == parts.length)
                break;

            item = parts[index++];
            item = item.trim();
            if (!item.equals("null")) {
                int leftNumber = Integer.parseInt(item);
                node.left = new TreeNode(leftNumber);
                nodeQueue.add(node.left);
            }

            if (index == parts.length)
                break;

            item = parts[index++];
            item = item.trim();
            if (!item.equals("null")) {
                int rightNumber = Integer.parseInt(item);
                node.right = new TreeNode(rightNumber);
                nodeQueue.add(node.right);
            }
        }
        return root;
    }

    public static void printTree(TreeNode root){
        if(root == null)
            return;
        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        List<List<String>> ret = new LinkedList<>();

        while(!queue.isEmpty()){
            for(int i = 0; i < queue.size(); i++){
                ArrayList<String > res = new ArrayList<>();
                TreeNode curNode = queue.removeFirst();
                res.add(String.valueOf(curNode.val));
                if(curNode.left != null) {
                    queue.add(curNode.left);
                    res.add(String.valueOf(curNode.left.val));
                } else
                    res.add("null");
                if(curNode.right != null) {
                    queue.add(curNode.right);
                    res.add(String.valueOf(curNode.right.val));
                } else
                    res.add("null");
                ret.add(res);
            }
        }
        for(List<String> item: ret)
            System.out.printf("%s -> %s, %s\n", item.get(0), item.get(1), item.get(2));
    }

    public static void main(String[] args) {
        Solution s = new Solution();
        TreeNode root = s.listToTreeNode(new String("1,2,3,4,null,6,null"));
        printTree(root);
    }
}
~~~
