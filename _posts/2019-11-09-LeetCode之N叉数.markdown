---
layout:      post
title:       "LeetCode之N叉数"
subtitle:    "LeetCode : /n-ary-tree"
author:      "Ashior"
header-img:  "img/post-bg-ccf.jpg"
catalog:     true
tags:
  - 工作
  - LeetCode
  - 算法
---

> 题目信息谷歌一下即可，也可通过 [LeetCodeN叉数](https://leetcode-cn.com/explore/learn/card/n-ary-tree/) 查看

[TOC]

----

## N-ary Tree Preorder Traversal

**源码**

我的递归代码：

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val;
    vector<Node*> children;

    Node() {}

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
public:
    void digui(Node* root, vector<int>& vec) {
        if (root == nullptr) {
            return ;
        }
        vec.push_back(root->val);
        for (auto a : root->children) {
            digui(a, vec);
        }
    }
    vector<int> preorder(Node* root) {
        vector<int> vec;
        digui(root, vec);
        return vec;
    }
};
```

迭代版本：

```cpp
class Solution {public:
    vector<int> preorder(Node* root) {
        stack<Node* > node_stack;
        Node* n = root;
        vector<int> result;
        if (root == NULL) {
            return result;
        }
        while (!node_stack.empty() || n)  {
            result.push_back(n->val);
            for (int i = n->children.size() - 1; i >= 0; i--) {
                node_stack.push(n->children[i]);
            }  
            n = NULL;
            if (!node_stack.empty()) {
                n = node_stack.top();
                node_stack.pop();
            }
        }
        return result;
    }
};
```

----

## N-ary Tree Postorder Traversal

**源码**

```cpp
class Solution {
public:
    void digui(Node* root, vector<int>& vec) {
        if (root == nullptr) {
            return ;
        }
        for (auto a : root->children) {
            digui(a, vec);
        }
        vec.push_back(root->val);
    }
    vector<int> postorder(Node* root) {
        vector<int> vec;
        digui(root, vec);
        return vec;
    }
};
```

----

## N叉树的层序遍历

**源码**

我的代码：

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val;
    vector<Node*> children;

    Node() {}

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
public:
    void digui(vector<Node*> vvv, vector<vector<int>>& vec, int index) {
        if (vvv.size() == 0) {
            return ;
        }
        // cout << vec.size() << " " << index << endl;
        if (vec.size() > index) {
            for (auto a : vvv) {
                vec[index].push_back(a->val);
            }   
        } else {
            vector<int> v;
            for (auto a : vvv) {
                v.push_back(a->val);
            }
            vec.push_back(v);
        }
        for (auto a : vvv) {
            digui(a->children, vec, index+1);
        }
    }
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>> vec;
        if (root == nullptr) {
            return vec;
        }
        vector<int> tmp;
        tmp.push_back(root->val);
        vec.push_back(tmp);
        digui(root->children, vec, 1);
        return vec;
    }
};
```

递归的代码优化：

```cpp
class Solution {public:
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>> ans;
        
        dfs(ans, root, 0);
        return ans;
    }
    void dfs(vector<vector<int>> &ans, Node* root, int level) {
        if(root) {
            if(ans.size() == level) {
                vector<int> tmp;
                ans.push_back(tmp);
            }
            ans[level].push_back(root->val);
            for(Node *child:root->children) {
                dfs(ans, child, level+1);
            }
        }
    }
};
```

迭代解决：

```cpp
class Solution {public:
    vector<vector<int>> levelOrder(Node* root) {
        vector<vector<int>> ans;
        if(!root) {
            return ans;
        }
        vector<Node*> currlevel = {root};
        
        while(currlevel.size()) {
            vector<Node*> nextlevel;
            vector<int> levelans;
            for(Node *node:currlevel) {
                levelans.push_back(node->val);
                // 将内容一次性考入至数组数组中
                nextlevel.insert(nextlevel.end(), node->children.begin(), node->children.end());
            }
            currlevel = nextlevel;
            ans.push_back(levelans);
        }
        
        return ans;
    }
};
```

**源码解析**

此题目用迭代其实更加方便。

`vector.insert()`函数有以下三种用法: 
1. 在指定位置loc前插入值为val的元素,返回指向这个元素的迭代器
2. 在指定位置loc前插入num个值为val的元素 
3. 在指定位置loc前插入区间[start, end)的所有元素 

----

## Maximum Depth of N-ary Tree

**源码**

```cpp
class Solution {
public:
    int maxDepth(Node* root) {
        if (root == nullptr) {
            return 0;
        } else {
            int maxd = 0;
            for (auto a : root->children) {
                maxd = max(maxd, maxDepth(a));
            }
            return maxd+1;
        }
    }
};
```
