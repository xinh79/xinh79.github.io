---
layout:      post
title:       "LeetCode之数组与字符串"
subtitle:    "LeetCode Vector String"
author:      "Ashior"
header-img:  "img/post-bg-ccf.jpg"
catalog:     true
tags:
  - 工作
  - LeetCode
  - 算法
---

> 题目信息谷歌一下即可，也可通过 [LeetCode数组与字符串](https://leetcode-cn.com/explore/learn/card/array-and-string/) 查看

> 解题思路主要是我的想法，源码和源码分析为我所提交的代码与LeetCode上面较好，通俗易懂的代码

----

## 寻找数组的中心索引

**解题思路**

失败了=。=

**源码**

```cpp
int pivotIndex(vector<int>& nums) {
    if( nums.size() == 0)
        return -1;
    
    int leftSum = 0, rightSum = 0;
    
    for( int i = 1; i < nums.size(); i++)
        rightSum += nums[i];
    
    if( leftSum == rightSum )
        return 0;
    
    for( int i = 1; i < nums.size(); i++) {
        rightSum -= nums[i], leftSum += nums[i - 1];
        if( rightSum == leftSum)
            return i;
    }
    
    return -1;
}
```

**源码分析**

设置两个向量`leftsum`, `rightsum`,作为比较，初始时，`leftsum =0`, `rightsum`为下标`1`到最后 的所有元素的和。如果二者相等，则返回下标`0`。否则开始遍历数组，对于每一个元素，`leftsum+=nums[i-1]`, `ightsum-=nums[i]`， 之后判断二者是否相等。

`rightSum -= nums[i], leftSum += nums[i - 1];`

这条语句为灵魂所在，其中`rightsum`是不断的减少，而实际移动的是`leftsum`。其中`i-1`开始递增，是由于如果有分割线，则必须会是在0之后，即`1, 2, 3...`。因为之前已经判断过，当对数组进行累加后的情况`if( leftSum == rightSum )`。

----

## 至少是其他数字两倍的最大数

**解题思路**

思路是对的，但是就是执行的思路模糊。整体思路十分简单，就是一次遍历，找出最大与第二大的数即可。

**源码**

```cpp
int dominantIndex(vector<int>& nums) {
	int num1 = -1;
	int num2 = -1;
	if (nums.size() == 0) {
		return -1;
	}
	int index = 0;
	for (int i = 0; i < nums.size(); i++) {
		if (nums[i] > num1) {
			num2 = num1;
			num1 = nums[i];
			index = i;
		} else if (nums[i] > num2) {
			num2 = nums[i];
		}
	}
	cout << endl << num1 << " " << num2 << endl;
	if (num1 >= num2*2) {
		return index;
	} else {
		return -1;
	}
}
```

**源码分析**

在`for`循环中就十分有灵性了，当出现一个比最大数还大的数字时，我们改变最大数的同时，将之前最大数赋值给第二大的数。

----

## 加一

**解题思路**

先通过对数组的入栈操作，对最后一位开始处理，之后再根据上一次处理的结果，来判断本次处理的流程。

**源码**

我的代码：

```cpp
vector<int> plusOne(vector<int>& digits) {
	vector<int> vec;
	if (digits.size() == 0) {
		return digits;
	} else if (digits[0] == 0 && digits.size() == 1) {
		vec.push_back(1);
		return vec;
	}
	stack<int> s;
	for (int i = 0; i < digits.size(); i++) {
		s.push(digits[i]);
	}
	bool flag = false;
	bool first = true;
	stack<int> tmps;
	while (!s.empty()) {
		int i = s.top();
		s.pop();
		if (flag)  {
			if (i+1 == 10) {
				tmps.push(0);
				flag = true;
			} else {
				tmps.push(i+1);
				flag = false;
			}
		} else {
			if (first) {
				if (i+1 == 10) {
					tmps.push(0);
					flag = true;
				} else {
					tmps.push(i+1);
					flag = false;
				}
			} else {
				tmps.push(i);
				flag = false;
			}   
		}
		first = false;
	}
	if (flag) {
		tmps.push(1);
	}
	while (!tmps.empty()) {
		vec.push_back(tmps.top());
		tmps.pop();
	}
	return vec;
}
```

高效代码：

```cpp
vector<int> plusOne(vector<int>& digits) {
	int N = (int)digits.size();
	for (int i = N - 1; i >= 0; i--) {
		if (digits[i] == 9) {
			digits[i] = 0;
		}
		else {
			digits[i]++;
			return digits;
		}
	}
	digits[0] = 1;
	digits.push_back(0);
	return digits;
}
```

**源码分析**

我的思路明显就是比较low的=。=以下是别人的源码。

其实很多时候只要判断一次，如果碰到了`9`，那么就需要`+1`操作，如果不是，则直接末尾`+1`返回即可。

----

## 对角线遍历

**解题思路**

根据问题，遍历其实就是两种方向，向上或者向下。再增加对三种边界条件的判断(向上时碰到右上角/上边界/右边界，向下时碰到左下角/左边界/下边界)。

**源码**

我的代码：

```cpp
vector<int> findDiagonalOrder(vector<vector<int>>& matrix) {
	int label = 1;
	vector<int> vec;
	if (matrix.size() == 0) {
		return vec;
	}
	int i = 0;
	int j = 0;
	bool flag = true;
	while(flag) {
		switch(label) {
			case 1:
				if (i == matrix.size()-1 && j == matrix[0].size()-1) {
					flag = false;
					vec.push_back(matrix[i][j]);
					break;
				}
				vec.push_back(matrix[i][j]);
				// 说明碰到了右上角 
				if ( i-1 < 0 && j+1 >= matrix[0].size() ) {
					i = 1;
					label = 2; 
				// 向上触顶 
				} else if (i-1 < 0) {
					j = j+1;
					label = 2;
				} else if (j+1 >= matrix[0].size()) {
					i = i+1;
					label = 2;
				} else {
					i = i-1;
					j = j+1;
				}
				break;
			case 2:
				if (i == matrix.size()-1 && j == matrix[0].size()-1) {
					flag = false;
					vec.push_back(matrix[i][j]);
					break;
				}
				vec.push_back(matrix[i][j]);
				if ( i+1 >= matrix.size() && j-1 < 0 ) {
					label = 1;
					j = 1;
				} else if (i+1 >= matrix.size()) {
					label = 1;
					j++;
				} else if (j-1 < 0) {
					label = 1;
					i++;
				} else {
					i++;
					j--;
				}
				break;
		}
	}
	return vec;
}
```

**源码解析**

注意边界判断即可。(这是我第一次一口气写完，然后在leetcode上面出现战胜100%的code`[76 ms]`)

----

## 螺旋矩阵

**解题思路**

和上一题一样，就是对边界的判断，新增一个方向判断。

**源码**

我的代码：

```cpp
vector<int> spiralOrder(vector<vector<int>>& matrix) {
	vector<int> vec;
	if (matrix.size() == 0) {
		return vec;
	}
	vector<vector<int>> visited(matrix.size(), vector<int>(matrix[0].size(), 1));
    int count = matrix.size() * matrix[0].size();
    int label = 1;
    int i = 0;
    int j = 0;
    bool flag = true;
    vec.push_back(matrix[0][0]);
    count--;
    visited[0][0] = 0;
    while (flag) {
    	switch(label) {
	    	case 1:
	    		if (count == 0) {
	    			flag = false;
	    			break;
				}
	    		if (j+1 >= matrix[0].size() || visited[i][j+1] == 0) {
	    			label = 2;
	    			break;
				} else {
					j++;
				}
	    		vec.push_back(matrix[i][j]);
	    		visited[i][j] = 0;
	    		count--;
	    		break;
	    	case 2:
	    		if (count == 0) {
	    			flag = false;
	    			break;
				}
	    		if (i+1 >= matrix.size() || visited[i+1][j] == 0) {
	    			label = 3;
	    			break;
				} else {
					i++;
				}
				vec.push_back(matrix[i][j]);
	    		visited[i][j] = 0;
	    		count--;
	    		break;
	    	case 3:
	    		if (count == 0) {
	    			flag = false;
	    			break;
				}
	    		if (j-1 < 0 || visited[i][j-1] == 0) {
	    			label = 4;
	    			break;
				} else {
					j--;
				}
				cout << "3: " << matrix[i][j] << endl;
				vec.push_back(matrix[i][j]);
	    		visited[i][j] = 0;
	    		count--;
	    		break;
	    	case 4:
	    		if (count == 0) {
	    			flag = false;
	    			break;
				}
	    		if (i-1 < 0 || visited[i-1][j] == 0) {
	    			label = 1;
	    			break;
				} else {
					i--;
				}
	    		vec.push_back(matrix[i][j]);
	    		visited[i][j] = 0;
	    		count--;
	    		break;
		}
	}
    return vec;
}
```

高效代码：

```cpp
vector<int> spiralOrder(vector<vector<int>>& matrix) {
	if(matrix.empty() || matrix[0].empty())return {};
	vector<int> res;
	int m = matrix.size(),n=matrix[0].size();
	//确定上下左右四条边的位置
	int up = 0,down = m -1,left = 0,right = n-1;
	while(true)
	{
		for(int i = left;i <= right; i++)res.push_back(matrix[up][i]);
		if(++up >down)break;
		for(int i = up;i <= down; i++)res.push_back(matrix[i][right]);
		if(--right < left) break;
		for(int i = right;i >= left;i--)res.push_back(matrix[down][i]);
		if(--down < up)break;
		for(int i = down; i >= up;i--)res.push_back(matrix[i][left]);
		if(++left >right) break;
	}
	return res;
}
```

**源码解析**

自己的代码就是死路直白，简单易懂，但是很多重复的内容，看下别人的源码吧。

一样是根据上下左右四个方向进行循环。不过作者利用了一点，就是每一次循环，相应的边界就缩减一圈，因此用上下或者左右的边界判断是否循环完毕。

----

## 杨辉三角

**解题思路**

根据数学规律结题即可。

**源码**

我的代码：

```cpp
vector<vector<int>> generate(int numRows) {
    vector<vector<int>> vec;
	if (numRows == 0) {
		return vec;
	}
	vector<int> v;
	v.push_back(1);
	vec.push_back(v);
	if (numRows == 1) {
		return vec;
	}
	v.push_back(1);
	vec.push_back(v);
	if (numRows == 2) {
		return vec;
	}
	vector<int> tmpv;
	for (int i = 3; i < numRows; i++) {
		tmpv.push_back(1);
		v = vec[i-2];
		for (int j = 0; j < v.size()-1; j++) {
			tmpv.push_back(v[j] + v[j+1]);
		}
		tmpv.push_back(1);
		vec.push_back(tmpv);
		tmpv.clear();
	}
	return vec;
}
```

高效代码：

```cpp
vector<vector<int>> generate(int numRows) {
	vector<vector<int>> obj(numRows);
	for(int i = 0; i < numRows; i ++) {
		obj[i].resize(i+1);
		obj[i][0] = 1;
		obj[i][i] = 1;
		for(int k =1; k< i; k++) {
			obj[i][k] = obj[i-1][k-1] + obj[i-1][k];
		}
	}
	return obj;
}
```

**源码解析**

他人的源码十分简练，而且不需要提前判断。根据杨辉三角性质，首先就是首位与末尾均为1。每一次循环，均比上一次的列`+1`，因此有`obj[i].resize(i+1);`重新分配大小，根据公式`obj[i][k] = obj[i-1][k-1] + obj[i-1][k];`计算结果。


----

## 反转字符串

**解题思路**


**源码**

```cpp
void reverseString(vector<char>& s) {
	int i = 0;
	int j = s.size() - 1;
	while ( i > j ) {
		swap(s[i++], s[j--]);
	}
}
```

**源码解析**

可以不需要`if`语句做边界判断，因为`j`会为`-1`，如果数组为空。

----

## 拆分数组 I

**解题思路**



**源码**

```cpp
int arrayPairSum(vector<int>& nums) {
 if (nums.size() == 0) {
  return 0;
 }
 int res = 0;
    sort(nums.begin(), nums.end());
    for (int i = 0; i < nums.size(); i+=2) {
     res = nums[i] + nums[i+1] + res;
 }
 return res;
}
```

**源码解析**


----

## 两数之和 II - 输入有序数组

**解题思路**

先采用二分法查找，找出离目标数最近，且不大于目标数的数。之后再对此下标之前的数据进行遍历。

**源码**

我的代码：

```cpp
vector<int> twoSum(vector<int>& numbers, int target) {
	vector<int> vec;
	if (numbers.size() == 0 || numbers[0] > target) {
		return vec;
	}
	int mid = numbers.size()/2;
	int left = 0;
	int right = numbers.size()-1;
	while (left <= mid) {
		if (numbers[mid] == target) {
			break;
		} else if (numbers[mid] > target) {
			right = mid-1;
			mid = (right + left)/2;
		} else {
			left = mid+1;
			mid = (right + left)/2;
		}
	}
	if (mid + 1 >= numbers.size()) {
		mid = numbers.size() - 1;
	} else {
		mid++;
	}
	for (int i = mid; i > 0; i--) {
		for (int j = 0; j < i; j++) {
			if (numbers[i] + numbers[j] == target) {
				vec.push_back(j+1);
				vec.push_back(i+1);
				return vec;
			}
		}
	}
	return vec;
}
```

高效代码：

```cpp
vector<int> twoSum(vector<int>& numbers, int target) {
	int slow = 0;
	int fast = numbers.size()-1;
	vector<int> res(2);
	while(slow < fast) {
		if(numbers[slow] + numbers[fast] == target) {
			res[0] = slow+1;
			res[1] = fast+1;
			break;
		} else if(numbers[slow] + numbers[fast] < target) {
			slow++;
		} else {
			fast--;
		}
	}
	return res;
}
```

**源码解析**

把问题想复杂了，一次遍历循环即可，思路不变，首尾相加，如果不想等，则判断是更大，还是更小，在决定是从左边递增还是右边递减。

----

## 移除元素

**解题思路**

由于最后返回的元素可以为任意顺序，我想对于找到的值从最后开始往前填充，避免元素的大量移动。

**源码**

我的代码：

```cpp
int removeElement(vector<int>& nums, int val) {
    if (nums.size() == 0) {
    	return 0;
	}
	int count = 0;
	for (int i = 0; i < nums.size() - count; i++) {
		if (nums[i] == val) {
			for (int j = nums.size() - 1 - count; j >= i; j--) {
				if (nums[j] != val) {
					swap(nums[i], nums[j]);
					count++;
					break;
				} else {
					count++;
				}
			}
		}
	}
}
```

高效代码：

```cpp
int removeElement(vector<int>& nums, int val) {
	int i,j=0;
	for(i=0;i<nums.size();i++) {
		if(nums[i]!=val) {
			nums[j]=nums[i];
			j++;
		}
	}
	return j;
}
```

**源码解析**

思路很简单，快慢指针，如果不想等，则同步进行，如果出现相等的情况，那么`j`的值就会比`i`慢一步。最后的返回结果也直接返回`j`即可。

----

## 最大连续1的个数

**解题思路**

模仿上一题，利用快慢指针。其实都不算是快慢指针，顺序遍历，碰到`1`就递增，非`1`清零，同时比较与所记录的最大值，以便更新。

**源码**

```cpp
int findMaxConsecutiveOnes(vector<int>& nums) {
    int max = 0;
    int j = 0;
    for (int i = 0; i < nums.size(); i++) {
    	if (nums[i] == 1) {
    		j++;
		} else {
			if (j > max) {
				max = j;
			}
			j = 0;
		}
	}
	if (j > max) {
		max = j;
	}
    return max;
}
```

----

## 长度最小的子数组

**解题思路**

尝试一下，先通过`sort`排序，之后再逐个增加，知道找到相匹配的。
大错特错，因为题目中只是找到`>=s`的连续数组个数，所以在这个过程中，并不需要排序，也不能对数组排序。

这道题目就是通过两层循环，然后不断累加，记录达到要求时的情况，之后再去相对较小的次数保存。

**源码**

我的代码：

```cpp
int minSubArrayLen(int s, vector<int>& nums) {
	if (nums.size() == 0) {
		return 0;
	}
	int minc = INT_MAX;
	int count = 0;
	int tmp = 0;
	for (int i = 0; i < nums.size(); i++) {
		if (nums[i] >= s) {
			return 1;
		}
		count = 1;
		tmp = nums[i];
		for (int j = i+1; j < nums.size(); j++) {
			if (tmp + nums[j] >= s) {
				count++;
				minc = min(minc, count);
				break;
			} else {
				tmp += nums[j];
				count++;
			}
		}
		if (tmp >= s) {
			minc = min(minc, count);
		}
	}
	if (minc > nums.size()) {
		return 0;
	} else {
		return minc;
	}
}
```

高效代码：

```cpp
int minSubArrayLen(int s, vector<int>& nums) {
	int n = (int)nums.size();
	int min_left = 0;
	int min_len = INT_MAX;
	int left = 0;
	int sum = 0;
	for(int i = 0; i < n; i++){
		sum += nums[i];
		while(sum >= s){
			if(i-left+1 < min_len){
				min_len = i - left+1;
				min_left = left;
			}
			sum -= nums[left];
			left++;
		}
	}
	return min_len == INT_MAX ? 0 : min_len;
}
```

**滑动窗口实现：**

```cpp
int minSubArrayLen(int s, vector<int>& nums) {
	if (nums.empty())
		return 0;
	// 滑动窗口
	int left = 0;
	int right = -1;
	int sum = 0;
	int minlen = INT_MAX;
	int len = nums.size();
	while (right < len){
		// 不断增加窗口大小，直到找到满足条件的连续序列
		while (sum < s && right < len){
			sum += nums[++right];
		}
		if (sum >= s){
			// 找到了满足条件的情况，更新minlen
			if ((right - left + 1) <= minlen)
				minlen = right - left + 1;
			// 左边开始移动，试图缩减窗口大小
			sum -= nums[left++];
		}
	}
	return minlen < len ? minlen : 0;
}
```

**源码解析**

首先我的想法是暴利破解，庆幸的是用`break`做了`剪枝`操作，不然肯定会超时。

要求是连续子数组，所以我们必须定义`i`，`j`两个指针，`i`向前遍历，`j` 向后遍历，相当与一个滑块，这样所有的子数组都会在`[i...j]`中出现，如果`nums[i..j]`的和小于目标值`s`，那么j向后移一位，再次比较，直到大于目标值`s`之后，`i`向前移动一位，缩小数组的长度。遍历到i到数组的最末端，就算结束了，如果不存在符合条件的就返回`0`。

查看题目，我们可以发现，最短的子数组有两个可能性：

1.  在最大数附近；
2.  在较大数附近；

也考虑到两种限制：

1.  最大值直接大于给定数值；
2.  所有数值相加小于给定数值；

针对以上，我们可以联想使用滑动窗口，即左右相同起点，右指针往右移直至累加值`sum > s`，再逐次将左指针右移，`sum`值递减，直到`sum < s`，再次移动右指针如此反复。期间，每当到达临界值的时候，记录最小长度。

上面的话看的有点晦涩，静下心来，看代码的注释，就会发现这就是一个十分简单的`滑动窗口`的实现。

双指针通常用于两个场景：`快慢指针`与`滑动窗口`。

----

## 旋转数组

**解题思路**

看了他人的解题思路，通过

**源码**

高效代码：

```cpp
void rotate(vector<int>& nums, int k) {
	if (nums.size() == 0) {
		return ;
	}
	k = k%nums.size();
	if (k == 0) {
		return;
	}
	reverse(nums.begin(), nums.end()-k);
	reverse(nums.end()-k, nums.end());
	reverse(nums.begin(), nums.end());
}
```

```cpp
class Solution {
public:
    //数组反转函数
    void reverseArray(vector<int>& array, int begin, int end)
    {
        int temp, tmp_end = end;
        for (int i = begin; i <= (begin + end)/2; i++)
        {
            temp = array[i];
            array[i] = array[tmp_end];
            array[tmp_end] = temp;
            tmp_end--;
        }
    }
    
    void rotate(vector<int>& nums, int k) {
       int len = nums.size();
        k %= len; 
        if(k == 0)
            return; 
        //使用自定义的反转函数
        reverseArray(nums, 0, len - k - 1);
        reverseArray(nums, len - k, len - 1);
        reverseArray(nums, 0, len - 1);
 
        //使用C++自带的反转函数
	    /*
		 * reverse(nums.begin(), nums.end() - k);
	     * reverse(nums.end() - k, nums.end());
	     * reverse(nums.begin(), nums.end());
		 */
	}
};
```

```cpp
void rotate(vector<int>& nums, int k) {
	int *a = new int[nums.size()];
	for(int i = 0; i < nums.size(); i++){
		a[(i+k) % nums.size()] = nums[i];
	}
	for(int j = 0; j < nums.size(); j++){
		nums[j] = a[j];
	}
}
```

**源码解析**

对于第一种方式，使用旋转时，不能使用`nums.begin()+k+1`，只能通过`nums.end()-k`，这是为了避免越界与判断旋转时出问题。主要是出在`+1`这里，需要警惕。

第二种方法为LeetCode中最快速的方法。但是没有满足空间复杂度为`O(1)`。通过`取模`操作，将元素赋值给辅助数组，再将数组赋值给`nums`。

----

## 杨辉三角 II

**解题思路**

没有想到很好的方法，中途还被写本子的事情给终止了很久。(都是借口=。=)

**源码**

```cpp
vector<int> getRow(int rowIndex) {
	vector<int> res(rowIndex + 1);
	res[0] = 1;
	for (int i = 1; i <= rowIndex; ++i) {
		for (int j = i; j >= 1; --j) {
			res[j] += res[j - 1];
		}
	}
	return res;
}
```

**源码解析**

根据`res[j] += res[j - 1];`计算出每一行的杨辉三角，然后再第下一行的时候，根据上一行的结果原地计算。

----

## 翻转字符串里的单词

**解题思路**

由于里面的单词并不需要翻转，尝试将读取字符串的操作，然后再将字符串进行连接操作，这时的连接是通过前向添加的操作完成。

**源码**

字符转字符串：

```cpp
const char c = 'a';
// 1.使用 string 的构造函数
string s(1,c);
// 2.声明string 后将char push_back
string s1;
s1.push_back(c);
// 3.使用stringstream
stringstream ss;
ss << c;
string str2 = ss.str();
// 注意 使用to_string 方法会转化为char对应的ascii码
// 原因是 to_string 没有接受char型参数的函数原型，有一个参数类型
// 为int 的函数原型，所以传入char型字符 实际是先将char 转化
// 为int 型的ascii 码，然后再转变为string
// 以下输出结果为 97
cout << to_string(c) << endl;
```

我的代码：

```cpp
string reverseWords(string s) {
	string str = "";
	// 去除首尾空格 
	if (!s.empty()) {
		s.erase(0, s.find_first_not_of(" "));
		s.erase(s.find_last_not_of(" ") + 1);
	}
	string tmp = "";
	for (int i = 0; i < s.length(); i++) {
		if (s[i] == ' ' && s[i-1] != ' ') {
			tmp = " " + tmp;
			str = tmp + str;
			tmp.clear();
			continue;
		}
		if (s[i] == ' ' && s[i-1] == ' ') {
			continue;
		}
		tmp.push_back(s[i]); 
	}
	str = tmp + str;
	return str;
}
```

简易代码：

```cpp
string reverseWords(string s) {
	//构造函数初始化
	stringstream o(s);
	string temp;
	string res;
	o>>res;
	while(o>>temp){
		res= temp+' '+ res;
	}
	return res;
}
```

高效代码：

```cpp
// way1:利用stringstream过滤空格；空间复杂度不是o(1)
string reverseWordsWay1(string s) {
	int n = s.size();
	vector<string> temp;
	stringstream strs(s);
	string str,result;
	while (strs >> str) {
		temp.push_back(str);
	}
	int lens = temp.size();
	for (int i = lens-1; i >= 0; i--) {
		result += temp[i]+" ";
	}
	//去掉多加的一个空格
	return result.size()? string(result.begin(), result.end() - 1):"";
}
// way2:先翻转整个单词，再翻转局部;空间复杂度为o(1)
string reverseWordsWay2(string s) {
	//翻转整体
	reverse(s.begin(), s.end());
	int start = 0, end = s.size() - 1;
	//找到首部第一个非空字符
	while (s[start] == ' ' && start <= end) {
		start++;
	}
	//找到尾部第一个非空字符
	while (s[end] == ' ' && end >= start) {
		end--;
	}

	if (start > end) {
		//特殊情况即字符串全为空字符
		return "";
	}
	// 翻转中间的单词
	for (int r = start; r <= end; r++) {
		while (s[r] == ' ' && r <= end) {
			r++;
		}
		int l = r;
		while (s[r] != ' ' && r <= end) {
			r++;
		}
		reverse(s.begin() + l, s.begin() + r);
		// important:不要忘记移动r
	}

	// 处理中间部分多于的空格
	int tail = start;
	for (int i = start; i <= end; i++) {
		if (s[i] == ' ' &&s[i + 1] == ' ') {
			continue;
		}
		s[tail++] = s[i];
	}
	return s.substr(start,tail-start);
}
```


**源码解析**

主要看上面的注释吧，已经很详细了。其实我还是更喜欢我的方法与最简单的方法，利用`stringstream`流。`stringstream`通常是用来做数据转换的，用于字符串与其他变量类型的转换，相比`c库`的转换，它更加安全，自动和直接。

1. 使用stringstream类前，先在头文件中添加 include<sstream>
2. stringstream对象的使用和cout对象的使用相同
3. 使用stringstream类前，先创建一个对象，并通过运算符“<<”将数据传给该对象
4. 使用时调用 [对象名].str()就行了
5. 实例：

```cpp
//定义当前帧 ,显然该变量不是字符型号，用stringstream转换格式
long currentFrame = 0;
stringstream str;
//将数据传给对象str
str << "pt" << currentFrame << ".jpg";
//将帧转成图片输出
imwrite("/home/image" + str.str(), frame);

int main() {
	string str1("How are you? 123 1 4.368");
    stringstream ss(str1);//构造函数初始化
    cout<<ss.str()<<endl;
    string str2;
    for(int i=0;i<3;i++) {
    	ss>>str2;
    	cout<<str2<<" ";
	}
	cout << endl; 
	int a;
	ss>>a;
	cout<<a<<endl;
	bool b;
	ss>>b;
	cout<<b<<endl;
	float c;
	ss>>c;
	cout<<c<<endl;	
	ss.clear();            // 注意，必须清空一次
	ss.str("I am fine!");  // 重新赋值
	while(ss>>str2) {      // 不断读取
		cout<<str2<<" ";
	}		
	return 0;
}
```

----

## 反转字符串中的单词 III

**解题思路**

做完了上一题，这一题就十分轻松了，向翻转整个字符串，之后再按照空格分块接入到新的字符串上面。

**源码**

```cpp
string reverseWords(string s) {
	reverse(s.begin(), s.end());
	string str;
	string tmp;
	for (int i = 0; i < s.size(); i++) {
		if (s[i] != ' ') {
			tmp.push_back(s[i]);
		} else {
			tmp.push_back(' ');
			str = tmp + str;
			tmp.clear();
		}
	}
	tmp.push_back(' ');
	str = tmp + str;
	str.erase(0, str.find_first_not_of(" "));
	str.erase(str.find_last_not_of(" ") + 1);
	return str;
}
```

----

## 删除排序数组中的重复项

**解题思路**

尝试的思路是利用快慢指针的思想：一个慢指针，指向不重复的数字(最后也是返回指针的值，一个正整数)；快指针则是遍历数组。当快指针指向的值不等于慢指针指向的下一个数值时，则表示不同需要进行替换操作。

**源码**

我的代码：

```cpp
int removeDuplicates(vector<int>& nums) {
	if (nums.size() == 0) {
		return 0;
	} else if (nums.size() == 1) {
		return 1;
	} 
	int low = 0;
	for (int i = 0; i < nums.size(); i++) {
		if ((nums[i] != nums[low])) {
			if ((low+1) != i) {
				swap(nums[i], nums[low+1]);
				low++;
			} else {
				low++;
			} 
		}
	}
	return low+1;
}
```

高效代码：

```cpp
int removeDuplicates(vector<int>& nums) {
	if(nums.size()==0)
		return 0;
	int fast=0,slow=0;
	for(int i=0;i<nums.size();i++) {
		if(nums[fast]!=nums[slow]) {
			slow++;
			nums[slow]=nums[fast];
		}
		fast++;
	}
	return slow+1;
}
```

**源码解析**

我的思路是对的，但是下面的写的更清晰、简洁，学习一下！

----

## 移动零

**解题思路**

尝试看下能否用快慢指针解决这个问题。一个顺序移动，一个记录0号出现的位置。
思来想去，还是失败了，最后觉得还是考虑交换吧，因为0元素后移，就表示非零元素前移。为了尽量减少移动次数，就是将需要移动的元素一次性移动至指定的位置，通过判断0元素的个数，则可以判断非零元素需要前移多少位。

**源码**

```cpp
void moveZeroes(vector<int>& nums) {
	int count = 0;
	for (int i = 0; i < nums.size(); i++) {
		if (nums[i] == 0) {
			count++;
		} else {
			swap(nums[i-count], nums[i]);
		}
	}
}
```
