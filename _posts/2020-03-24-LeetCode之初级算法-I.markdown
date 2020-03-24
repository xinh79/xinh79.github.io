---
layout: post
title: "LeetCode之初级算法-I"
subtitle: "LeetCode : questions-easy"
author: "Ashior"
header-img: "img/post-bg-ccf.jpg"
catalog: true
tags:
  - 工作
  - LeetCode
  - 算法
---

> 重启秋招火种计划。笔者看完了剑指 offer 后，现在开始对着他进行刷题，刷题（LeetCode）过程中有与书本中更好的解法，将会在本篇博客中列出，给出具体思路。并非完全按照 leetcode 题目路径，中间会穿插一些与书中类似的题目。参考原网址：https://leetcode-cn.com/explore/interview/card/top-interview-questions-easy/

---

<!-- TOC -->

- [丑数](#丑数)
- [机器人的运动范围](#机器人的运动范围)
- [割绳子](#割绳子)
- [1 的个数](#1-的个数)
- [股票最大利润](#股票最大利润)
- [旋转图像](#旋转图像)
- [删除链表中的指定节点](#删除链表中的指定节点)
- [删除排序数组中的重复项](#删除排序数组中的重复项)
- [旋转数组](#旋转数组)
- [存在重复](#存在重复)
- [只出现一次的数字](#只出现一次的数字)
- [两个数组的交集 II](#两个数组的交集-ii)
- [加一](#加一)
- [移动零](#移动零)
- [两数之和](#两数之和)
- [有效的数独](#有效的数独)
- [字符串中的第一个唯一字符](#字符串中的第一个唯一字符)
- [有效的字母异位词](#有效的字母异位词)
- [验证回文字符串](#验证回文字符串)
- [字符串转整数(atoi)](#字符串转整数atoi)
- [实现 strStr()](#实现-strstr)
- [最长公共前缀](#最长公共前缀)

<!-- /TOC -->

---

## 丑数

在书本中的一个解释可以套用，即对于一个丑数，势必是原始丑数 1 经过若干次 2、3、5 相乘最后得到的，要获取第 N 个丑数，我们可以从 1 开始，将循环 N 次，通过 3 个“指针”分别指向数组的数组，每次将最小的丑数赋值在数组中，并且对应的丑数指针移动一位，表示下一次相乘的过程就从此处开始。

原书中一种暴力解法为初始解法，请参考笔者之前的博客，还是需要了解的，毕竟此方法是基于暴力方法的衍生。

```cpp
class Solution {
public:
    int nthUglyNumber(int n) {
        if (n == 0) return n;
        vector<int> ugly(n, 0);
        ugly[0] = 1;
        int i = 0;
        int j = 0;
        int k = 0;
        // 每一次循环确定下一个丑数的值
        for (int index = 1; index < n; index++) {
            // 选择最小的一个丑数
            int tmp = min(ugly[i]*2, min(ugly[j]*3, ugly[k]*5));
            // 比较得出最小的丑数在哪个位置，然后填充
            if (tmp == ugly[i]*2) i++;
            if (tmp == ugly[j]*3) j++;
            if (tmp == ugly[k]*5) k++;
            ugly[index] = tmp;
        }
        return ugly[n-1];
    }
};
```

---

## 机器人的运动范围

比矩阵中的路径更简单，就是普通的 DFS 遍历，多一个限制条件，判断各位数之和是否超过 k 即可。剩下的就是判断是否满足遍历条件即可，满足则设置访问位后直接增加 1 并开始进入递归。

```cpp
class Solution {
public:
    bool judge(int row, int col, int k) {
        int res = 0;
        while (row != 0) {
            res += row % 10;
            row /= 10;
        }
        while (col != 0) {
            res += col % 10;
            col /= 10;
        }
        return res <= k ? true : false;
    }
    int help(int m, int n, int k, int i, int j, vector<vector<int>> &visited) {
        if (i < m && i >=0 && j < n && j >=0 && visited[i][j] != 1 && judge(i, j, k)) {
            visited[i][j] = 1;
            return 1 + help(m, n, k, i+1, j, visited)
                        + help(m, n, k, i, j+1, visited)
                        + help(m, n, k, i, j-1, visited)
                        + help(m, n, k, i-1, j, visited);
        } else {
            return 0;
        }
    }
    int movingCount(int m, int n, int k) {
        vector<vector<int>> visited(m, vector<int>(n, 0));
        return help(m, n, k, 0, 0, visited);
    }
};
```

---

## 割绳子

这个问题就是需要明白，每个割的绳子长度为 3 时，数值最大，如果小于 3 就是割成一个 1 和一个 2 即可。

```cpp
class Solution {
public:
    int cuttingRope(int n) {
        if (n == 1 || n == 2) {
            return 1;
        }
        if (n == 3) {
            return 2;
        }
        int sum = 1;
        while (n > 4) {
            sum *= 3;
            n -= 3;
        }
        return sum * n;
    }
};
```

---

## 1 的个数

一个数 n 与一个比它小 1 的数（n−1）进行与运算（&）之后，得到的结果会消除 n 中最低位的 1。

```cpp
int BitCount2(unsigned int n) {
    unsigned int c =0 ;
    for (c =0; n; ++c) {
        n &= (n -1) ; // 清除最低位的1
    }
    return c ;
}
```

---

## 股票最大利润

如果是剑指 offer 上面的题目，那么主要是记录当前位置 i 之前的最小值，然后依次将当前值，减去最小值，获得最大的正向差值。因为这个原题只要求购买一次股票，即买卖一次，如果像 leetcode 中可以多次买卖，求最大利润时，有提出用贪心策略，每次遇到比前一天价格更高时，就在前一天买入卖出。

剑指 offer 原题：

```cpp
int maxProfit(vector<int>& prices) {
    if (prices.size() == 0) return 0;
    int min = prices[0];
    int length = prices.size();
    int res = 0;
    for (int i = 1; i < prices.size(); i++) {
        min = prices[i] < min ? prices[i] : min;
        if (res < prices[i]-min) res = prices[i] - min;
    }
    return res;
}
```

一次股市可以多次买卖，采用贪心的策略，每当有利润时就抛售股票。

```cpp
int maxProfit(vector<int>& prices) {
    int min = INT_MAX;
    int res = 0;
    for (int i = 1; i < prices.size(); i++) {
        if (prices[i] < min) min = prices[i];
        if (prices[i] - prices[i-1] > 0) {
            res += prices[i] - prices[i-1];
        }
    }
    return res;
}
```

股票问题有很多变种，上面两种一种是只交易一次，另外一种是可以交易无数次。

[leetcode-一个通用方法团灭 6 道股票问题](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/solution/yi-ge-tong-yong-fang-fa-tuan-mie-6-dao-gu-piao-wen/)

---

## 旋转图像

矩阵旋转 90°，先将对角线的元素对调，再对调每一列的元素即可。注意对角线元素的下标求解位置。

```cpp
void rotate(vector<vector<int>>& matrix) {
    int length = matrix.size();
    // 交换对角线的元素
    for (int i = 0; i < length; i++) {
        for (int j = 0; j < length - i; j++) {
            int tmp = matrix[i][j];
            matrix[i][j] = matrix[length-j-1][length-i-1];
            matrix[length-j-1][length-i-1] = tmp;
        }
    }
    // 交换每一列的元素
    for (int i = 0; i < length/2; i++) {
        for (int j = 0; j < length; j++) {
            int tmp = matrix[i][j];
            matrix[i][j] = matrix[length - i - 1][j];
            matrix[length - i - 1][j] = tmp;
        }
    }
}
```

---

## 删除链表中的指定节点

唯一的困惑可能就是对于传入的参数只有一个，这样我们可以复制下一个节点，然后再将下一个节点删除。

```cpp
void deleteNode(ListNode* node) {
    if (node->next == nullptr) {
        delete node;
        node = nullptr;
    }
    ListNode* tmp = node->next;
    node->val = node->next->val;
    node->next = node->next->next;
    tmp->next = nullptr;
    delete tmp;
    tmp = nullptr;
}
```

---

## 删除排序数组中的重复项

由于是排序的数组，利用两个下标移动，index 表示每次移动的都是不一样的数字，如果一样的交换下一个位置。

```cpp
int removeDuplicates(vector<int>& nums) {
    // int res = 0;
    int index = 0;
    for (int i = 1; i < nums.size(); i++) {
        if (nums[i] != nums[index]) {
            swap(nums[++index], nums[i]);
        }
    }
    return nums.size() == 0 ? 0 : index+1;
}
```

---

## 旋转数组

就是将数组元素往后推 K 个位置，从而获得新的数组，反转三次即可。

```cpp
void rotate(vector<int>& nums, int k) {
    int length = nums.size();
    k = k % length;
    reverse(nums.begin(), nums.begin()+length-k);
    reverse(nums.begin()+length-k, nums.end());
    reverse(nums.begin(), nums.end());
}
```

---

## 存在重复

1. 可以利用额外空间，使用 set 数据结构去重
2. 创建一个很大很大的数组，直接将数据打在对应的下标上面，如果相同则返回 true，否则返回 false
3. 先对其进行排序，然后再遍历相邻的数据是否相等

```cpp
bool containsDuplicate(vector<int>& nums) {
    set<int> s;
    for (int i = 0; i < nums.size(); i++) {
        if (s.find(nums[i]) == s.end()) {
            s.insert(nums[i]);
        } else {
            return true;
        }
    }
    return false;
}
```

---

## 只出现一次的数字

```cpp
int singleNumber(vector<int>& nums) {
    int length = nums.size();
    if (length <= 1) return length == 0 ? -1 : nums[0];
    int res = nums[0];
    for (int i = 1; i < length; i++) {
        res ^= nums[i];
    }
    return res;
}
```

---

## 两个数组的交集 II

由于两个数组没有排序，所以通过一个 map 来进行计数操作。

```cpp
vector<int> intersect(vector<int>& nums1, vector<int>& nums2) {
    map<int, int> m;
    vector<int> res;
    for (int i = 0; i < nums1.size(); i++) {
        m[nums1[i]]++;
    }
    for (int i = 0; i < nums2.size(); i++) {
        if (m[nums2[i]] >= 1) {
            res.push_back(nums2[i]);
            m[nums2[i]]--;
        }
    }
    return res;
}
```

---

## 加一

原本的思想是将数组先逆序，然后逐位增加判断。其实下面这个方法就是将最后一个加 1 后判断，最后依次判断，如果最后 index 小于 0 了，则最后一位增加 0，然后把首位的 0 设置为 1.

```cpp
vector<int> plusOne(vector<int>& digits) {
    int length = digits.size();
    int index = length - 1;

    if(length == 0){
        return digits;
    }

    digits[index] += 1;

    while(digits[index] == 10){
        digits[index] = 0;
        index--;
        if(index < 0){
            break;
        }
        digits[index] += 1;
    }

    if(index < 0){
        digits.push_back(0);
        digits[0] = 1;
    }

    return digits;
}
```

---

## 移动零

由于不能修改元素的相对位置，所以不能使用 partition 的方法。利用元素两个指针移动。

```cpp
void moveZeroes(vector<int>& nums) {
    int index = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (nums[i] != 0) {
            swap(nums[index++], nums[i]);
        }
    }
}
```

---

## 两数之和

对于有序的数组，可以使用两个下标进行操作。

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    vector<int> res;
    int right = nums.size()-1;
    int left = 0;
    while (left < right) {
        if (nums[left] + nums[right] == target) {
            res.push_back(left);
            res.push_back(right);
            return res;
        } else if (nums[left] + nums[right] > target) {
            right--;
        } else {
            left--;
        }
    }
    return res;
}
```

对于无序的数组，可以通过 map 进行记录数据，然后找到对应的负值即可。

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    vector<int> res;
    map<int, int> m;
    for (int i = 0; i < nums.size(); i++) {
        m[nums[i]] = i;
    }
    for (int i = 0; i < nums.size(); i++) {
        // 还需要保证不是同一个数组
        if (m.find(target - nums[i]) != m.end() && m[target - nums[i]] != i) {
            res.push_back(i);
            res.push_back(m[target - nums[i]]);
            break;
        }
    }
    return res;
}
```

---

## 有效的数独

判断每一行，每一列，每一格中的数据。

```cpp
bool isValidSudoku(vector<vector<char>>& board) {
    // 表示每一行，然后第列表示对应的数字，比如row[2][3]=1表示第2行，3存在。以此列推。
    vector<vector<int>> row(board.size(), vector<int>(board[0].size(), 0));
    vector<vector<int>> col(board.size(), vector<int>(board[0].size(), 0));
    vector<vector<int>> cell(board.size(), vector<int>(board[0].size(), 0));
    for (int i = 0; i < 9; i++) {
        for (int j = 0; j < 9; j++) {
            if (board[i][j] != '.') {
                int index = board[i][j]-'1';
                if (row[i][index] == 1 || col[j][index] == 1 || cell[3*(i/3)+j/3][index] == 1) {
                    return false;
                }
                row[i][index] = 1;
                col[j][index] = 1;
                cell[3*(i/3)+j/3][index] = 1;
            }
        }
    }
    return true;
}
```

---

## 字符串中的第一个唯一字符

没啥说的，通过 ASCII 码记录其出现的值，遍历两遍字符串即可。

```cpp
int firstUniqChar(string s) {
    int c[256] = {0};
    for (int i = 0; i < s.length(); i++) {
        c[s[i]]++;
    }
    for (int i = 0; i < s.size(); i++) {
        if (c[s[i]] == 1) {
            return i;
        }
    }
    return -1;
}
```

---

## 有效的字母异位词

有效的字母用过 map 来标识，技术，同时需要注意不同长度的字符串一定不是异位词。

```cpp
bool isAnagram(string s, string t) {
    map<char, int> m;
    if (s.length() != t.length()) return false;
    for (int i = 0; i < s.size(); i++) {
        m[s[i]]++;
    }
    for (int i = 0; i < t.size(); i++) {
        if (m.find(t[i]) != m.end() && m[t[i]] > 0) {
            m[t[i]]--;
        } else {
            return false;
        }
    }
    return true;
}
```

---

## 验证回文字符串

```cpp
bool isPalindrome(string s) {
    string str = "";
    for (int i = 0; i < s.size(); i++) {
        if ((s[i] >= 'a' && s[i] <= 'z') || (s[i] >= '0' && s[i] <= '9')) {
            str += s[i];
        } else if (s[i] >= 'A' && s[i] <= 'Z') {
            str += (s[i]-'A'+'a');
        }
    }
    int left = 0;
    int right = str.size()-1;
    while (left <= right) {
        if (str[left++] != str[right--]) return false;
    }
    return true;
    }
```

---

## 字符串转整数(atoi)

提高代码抠边界的能力。需要考虑的是首先取出空格，然后判断第一个非空格字母是数字还是加号或者减号，亦或是字母，之后就是获取字符串，然后对字符串进行操作，记得选用 double，哪怕是 unsigned long long 都有问题，领扣这个系统有毛病。

```cpp
int myAtoi(string str) {
    int res = 0;
    if (str.size() == 0 || (str[0] >= 'a' && str[0] <= 'Z')) {
        return 0;
    }
    int left = 0;
    int flag = 1;
    int space = 0;
    while (space < str.size()) {
        if (str[space] == ' ') {
            space++;
        } else {
            break;
        }
    }
    left = space;
    if (str[left] == '-') {
        flag = -1;
        left++;
    } else if (str[left] == '+') {
        flag = 1;
        left++;
    } else if (str[0] >= 'a' && str[0] <= 'Z') {
        return 0;
    }
    string sss = "";
    while (left < str.size()) {
        if (str[left] >= '0' && str[left] <= '9') {
            sss = sss + str[left];
            left++;
        } else {
            break;
        }
    }
    double l = 0;
    left = 0;
    while (left < sss.size()) {
        l = l*10 + sss[left++]-'0';
    }
    if (flag == 1) {
        return l > INT_MAX ? INT_MAX : l;
    }
    if (l == 0) return 0;
    return -l < INT_MIN ? INT_MIN : -l;
}
```

---

## 实现 strStr()

这一题其实就是实现 KMP 算法。具体的话很久没写忘了，看了之前的[笔记](https://xinh79.github.io/2019/12/21/左程云算法讲解笔记-VII)，慢慢想起来了，熟能生巧啊~

```cpp
int* getnext(string& needle) {
    int *next = (int*)malloc(sizeof(int)*needle.size());
    if (needle.size() == 1) {
        next[0] = -1;
        return next;
    }
    next[0] = -1;
    next[1] = 0;
    int cn = 0;
    int pos = 2;
    while (pos < needle.size()) {
        if (needle[pos-1] == needle[cn]) {
            next[pos++] = ++cn;
        } else if (cn > 0) {
            cn = next[cn];
        } else {
            next[pos++] = 0;
        }
    }
    return next;
}
int strStr(string haystack, string needle) {
    if (needle.size() == 0) {
        return 0;
    }
    int *next = getnext(needle);

    int index1 = 0;
    int index2 = 0;

    while (index1 < haystack.size() && index2 < needle.size()) {
        if (haystack[index1] == needle[index2]) {
            index1++;
            index2++;
        } else if (next[index2] == -1) {
            index1++;
        } else {
            index2 = next[index2];
        }
    }
    free(next);
    return index2 == needle.size() ? index1 - index2 : -1;
}
```

---

## 最长公共前缀

纵向比较字符串。

```cpp
string longestCommonPrefix(vector<string>& strs) {
    string res = "";
    if (strs.size() == 0) {
        return res;
    }
    for (int i = 0; i < strs[0].size(); i++) {
        char c = strs[0][i];
        for (int j = 1; j < strs.size(); j++) {
            if(i >= strs[j].size() || strs[j][i] != c) {
                return res;
            }
        }
        res += c;
    }
    return res;
}
```
