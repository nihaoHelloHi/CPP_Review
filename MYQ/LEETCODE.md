# 动态规划
动态规划是解决最优化和计数问题的核心方法，其核心在于**状态定义**与**状态转移**。以下是主流动态规划题型的分类及解题思路框架，结合示例说明如何系统性思考：

---

### 一、分类体系与解题范式
#### 1. 线性序列问题（数组/字符串）
**典型问题**：最长递增子序列(LIS)、最大子数组和、编辑距离  
**状态定义**：  
- `dp[i]` 表示以第i个元素**结尾**时的最优解  
**转移思路**：  
```python
# 最长递增子序列
dp[i] = max(dp[j] + 1 for j in 0..i-1 if nums[j] < nums[i])
```

#### 2. 背包问题（组合优化）
**分类**：  
- 0-1背包：物品不可分割（选/不选）  
- 完全背包：物品无限使用  
- 多维背包：多个容量限制  
**状态定义**：  
- `dp[i][w]` 前i个物品在容量w时的最大价值  
**转移方程**：  
```python
# 0-1背包
dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])
```

#### 3. 矩阵路径问题（二维网格）
**典型问题**：最小路径和、不同路径  
**状态定义**：  
- `dp[i][j]` 表示到达网格(i,j)时的最优解  
**转移特点**：  
```python
# 最小路径和
dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
```

#### 4. 区间动态规划（分段处理）
**典型问题**：矩阵链乘法、回文分割  
**状态定义**：  
- `dp[i][j]` 表示区间[i,j]的最优解  
**转移模式**：  
```python
# 矩阵链乘法
dp[i][j] = min(dp[i][k] + dp[k+1][j] + cost for k in i..j-1)
```

#### 5. 状态机模型（多状态转换）
**典型问题**：股票买卖系列、打家劫舍  
**状态定义**：  
- `dp[i][state]` 第i天处于state状态的最优值  
**转移示例**：  
```python
# 股票买卖（冷冻期）
dp[i][hold] = max(dp[i-1][hold], dp[i-1][cooldown] - price)
dp[i][sold] = dp[i-1][hold] + price
dp[i][cooldown] = max(dp[i-1][cooldown], dp[i-1][sold])
```

#### 6. 树形动态规划（后序遍历）
**典型问题**：二叉树中的最大路径和、监控二叉树  
**状态定义**：  
- 通常返回元组包含多个状态（如选/不选当前节点）  
**转移示例**：  
```python
def dfs(root):
    if not root: return (0, 0)
    left = dfs(root.left)
    right = dfs(root.right)
    select = root.val + left[1] + right[1]
    not_select = max(left) + max(right)
    return (select, not_select)
```

---

### 二、通用解题步骤
1. **识别问题特征**  
   - 求最值/计数问题  
   - 决策过程有阶段性  
   - 子问题重叠（暴力递归存在重复计算）

2. **定义状态**  
   - 明确dp数组的物理含义  
   - 维度选择：时间步长、空间位置、物品索引、状态标记  
   - 示例：股票问题中的`dp[i][持有状态][交易次数]`

3. **推导转移方程**  
   - 分析最后一步的决策选择  
   - 考虑所有可能的前驱状态  
   - 使用数学表达式描述关系

4. **确定边界条件**  
   - 初始状态的赋值（如dp[0]）  
   - 无效状态的标记（如用-∞表示不可能情况）

5. **空间优化**  
   - 滚动数组：当当前状态仅依赖前1~2个状态时  
   - 维度压缩：如背包问题降维时需反向遍历

---

### 三、经典题型强化训练
| 题型                | 关键训练点                  | 代表题目                         |
|---------------------|---------------------------|--------------------------------|
| 路径计数            | 多维状态初始化            | 不同路径（有障碍物版）          |
| 字符串匹配          | 空串处理技巧              | 正则表达式匹配                  |
| 完全背包            | 组合与排列的区别          | 零钱兑换I/II                   |
| 状态压缩            | 位运算表示状态            | 最短哈密顿路径                  |
| 树形DP              | 后序遍历与状态返回        | 二叉树监控                      |

---

### 四、避坑指南
1. **过度设计状态**  
   - 错误：股票问题中冗余记录交易价格  
   - 正确：只需记录持有状态和剩余交易次数

2. **混淆状态转移方向**  
   - 错误：背包问题正向遍历时重复计数  
   - 正确：降维后需反向遍历容量

3. **忽视空间优化陷阱**  
   - 错误：滚动数组未正确维护历史状态  
   - 正确：二维数组优先实现，优化前验证正确性

通过系统性分类训练，配合LeetCode经典题（如72-编辑距离、121-股票买卖、322-零钱兑换），可快速建立动态规划解题直觉。建议从线性结构入手，逐步过渡到状态机等复杂模型。



# 异或类题目
在异或类算法问题中，常常需要理解异或（XOR）操作的性质和应用。以下是一些常见的思路和算法技巧，用于解决如“数组中任意两个数的最大异或值”等类型的题目。

### 1. **异或的性质**
首先，理解异或操作的性质是非常重要的。异或操作具有以下特点：

- **自反性**：`a ^ a = 0`。
- **交换性和结合性**：`a ^ b = b ^ a`，`(a ^ b) ^ c = a ^ (b ^ c)`。
- **与 0 的运算**：`a ^ 0 = a`。
- **二进制的按位运算**：对于二进制数，`a ^ b` 会对每一位进行比较：如果两位相同为 0，不同为 1。

### 2. **常见的异或类问题：**
比如求数组中任意两个数的最大异或值，或数组中与某个数异或后值最大的数等问题，常常通过 **前缀树（Trie）** 或 **贪心策略** 来求解。

---

### 3. **数组中任意两个数的最大异或值**

**题目概述：**
给定一个整数数组 `nums`，找到数组中任意两个元素的最大异或值。

#### **基本思路：**
1. **暴力解法**：直接枚举所有数对，计算每一对的异或值，返回最大的异或值。
   - 时间复杂度：`O(n^2)`，空间复杂度：`O(1)`。

2. **Trie（前缀树）优化法**：
   - 异或的最大值可以通过数的二进制位来逐位计算。由于异或是按位操作，因此可以通过将数字插入到 Trie 树中，逐位地查找异或最大的匹配。
   
   - **核心思想**：通过构建一个 Trie 来保存数组中每个数字的二进制表示。然后，依次计算每个数字和当前 Trie 中其他数字的最大异或值。
     - 从高位到低位遍历数字，每次尝试选择与当前位不同的值（来获得更大的异或值）。
   
   - **时间复杂度**：`O(n * L)`，其中 `n` 是数组的大小，`L` 是数字的位数（通常 `L` = 32）。
   
#### **实现**：
```cpp
class TrieNode {
public:
    TrieNode* children[2];
    TrieNode() {
        children[0] = children[1] = nullptr;
    }
};

class Solution {
public:
    int findMaximumXOR(vector<int>& nums) {
        TrieNode* root = new TrieNode();
        int maxXor = 0;

        for (int num : nums) {
            TrieNode* node = root;
            int currentXor = 0;
            for (int i = 31; i >= 0; --i) {  // 从高位到低位
                int bit = (num >> i) & 1;
                int oppositeBit = bit ^ 1;
                if (node->children[oppositeBit]) {
                    currentXor |= (1 << i);  // 如果存在相反的位，就更新最大异或值
                }
                if (!node->children[bit]) {
                    node->children[bit] = new TrieNode();
                }
                node = node->children[bit];
            }
            maxXor = max(maxXor, currentXor);
        }

        return maxXor;
    }
};
```

#### **解释**：
- `TrieNode` 是一个结构体，用于存储每一位（0 或 1）以及两个子节点。
- 对于每一个数字，我们从其最高位到最低位依次插入到 Trie 中。同时，在插入时，我们尝试选择与当前位相反的位，这样就能获取最大的异或值。
- 最终的 `maxXor` 变量就是我们所求的最大异或值。

#### **复杂度分析**：
- **时间复杂度**：`O(n * L)`，其中 `n` 是数组的大小，`L` 是数字的位数。
- **空间复杂度**：`O(n * L)`，存储每个数字的二进制位。

是的，**数组中任意两个数的最大异或值** 这道题可以使用 **位运算 + 哈希集合（unordered_set）** 来解决，且时间复杂度可以降到 **O(n)**。

---

### **思路：贪心 + 位运算 + 哈希集合**
核心思路是：
1. **从最高位到最低位贪心地确定异或值的每一位**：
   - 我们从**最高位**（第 31 位，对于 32 位整数）开始尝试，逐步确定最大异或值的每一位。
   - 每次尝试把该位设为 `1`，并检查是否可以找到两个数使其异或结果等于该值。
   
2. **利用集合存储前缀**：
   - 记录所有数字在当前位上的前缀（前 `k` 位）到一个哈希集合 `prefixSet` 中。
   - 遍历所有前缀，看是否有两个前缀可以形成目标异或值。

---

### **算法步骤**
1. 初始化变量 `maxXor = 0`，表示当前已确定的最大异或值。
2. 依次遍历整数的每一位，从最高位到最低位（从 31 位到 0 位）。
3. 在每一位 `k`，将当前可能的最大异或值 `candidate = maxXor | (1 << k)`。
4. 遍历数组，计算所有数在前 `k` 位的前缀，并存入 `prefixSet`。
5. 在 `prefixSet` 中查找，是否存在两个前缀 `prefixA` 和 `prefixB`，使 `prefixA ^ prefixB == candidate`。
   - 如果存在，说明该位可以是 `1`，更新 `maxXor = candidate`。
   - 否则，该位保持 `0`。

---

### **代码实现**
```cpp
class Solution {
public:
    int findMaximumXOR(vector<int>& nums) {
        int maxXor = 0, mask = 0;
        
        for (int i = 31; i >= 0; --i) {  // 从最高位开始
            mask |= (1 << i); // 更新掩码 mask，用于提取前缀
            unordered_set<int> prefixSet;

            // 计算所有数的前缀
            for (int num : nums) {
                prefixSet.insert(num & mask);  // 提取 num 的前 i+1 位前缀
            }

            int candidate = maxXor | (1 << i); // 目标异或值，尝试让当前位为 1

            // 在哈希集合中查找是否有两个前缀可以组成 candidate
            for (int prefix : prefixSet) {
                if (prefixSet.count(prefix ^ candidate)) {
                    maxXor = candidate; // 成功找到，更新 maxXor
                    break;
                }
            }
        }
        
        return maxXor;
    }
};
```

---

### **复杂度分析**
- **时间复杂度**：`O(32 * n) ≈ O(n)`，最多遍历 32 位，每次遍历 `n` 个元素。
- **空间复杂度**：`O(n)`，使用集合存储 `n` 个前缀。

---

### **解题思路解析**
1. **mask 作用**
   - `mask |= (1 << i)` 用于保留当前最高 `i+1` 位，忽略低位。
   - 例如，`i=30` 时，mask 为 `0b1100000000000000000000000000000`。

2. **候选最大异或值**
   - 设 `maxXor = 1010xxxx`（假设前面确定的部分）。
   - 在第 `k` 位，尝试让它变成 `1`，所以 `candidate = maxXor | (1 << k)`。

3. **贪心检查是否能实现**
   - 遍历所有前缀 `prefix`，如果存在 `prefixA` 和 `prefixB`，使 `prefixA ^ prefixB = candidate`，说明该位可以取 `1`，更新 `maxXor`。


---

### **总结**
- **贪心策略**：从高位到低位逐步构造最大异或值。
- **利用集合**：存储前缀，减少遍历次数，降低时间复杂度。
- **比 Trie 方法更简洁**，但 Trie 方法在处理大量数据时可能更稳定。

---

这种 **位运算 + 哈希集合** 的技巧在 XOR 相关的题目（如“最大异或子数组”）中也可以应用。


---

### 4. **其他与异或相关的技巧**

1. **与某个数异或后的最大值**：
   - 给定一个整数数组 `nums` 和一个整数 `x`，要求找到数组中与 `x` 异或后值最大的数。
   - 类似的方法也可以用 Trie 树来求解，使用与 `x` 异或的每一位来选择匹配最大值。

2. **异或前缀和**：
   - 有时需要查询数组前缀的异或值，可以用异或前缀和的技巧。定义 `prefixXor[i]` 为 `nums[0] ^ nums[1] ^ ... ^ nums[i]`。利用前缀和可以很方便地求出任意区间 `[i, j]` 的异或值：`prefixXor[j] ^ prefixXor[i-1]`。

3. **子数组异或和为零**：
   - 可以使用哈希表记录异或前缀和，在遇到相同的异或前缀时，说明区间的异或和为零。

---

### 总结：
- 异或类问题往往可以通过**贪心算法**、**前缀树（Trie）**、或者**异或前缀和**等技巧来求解。
- 了解异或的性质和操作，能帮助我们更好地设计出高效的算法来解决这些问题。

## 强数对的最大异或值
> https://leetcode.cn/problems/maximum-xor-of-two-numbers-in-an-array/solutions/2511644/tu-jie-jian-ji-gao-xiao-yi-tu-miao-dong-1427d

# 集合、位运算
> https://leetcode.cn/discuss/post/3571304/cong-ji-he-lun-dao-wei-yun-suan-chang-ji-enve/
> 


# 矩阵类问题
## 旋转矩阵类
> 一次旋转完成一轮(上下左右，或是一周)，使用top,bottom,left,right标定边界


# 字典树trie

# 排序
## 一比较排序
### **选择排序**
``` cpp
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        int curmax = 0;
        for (int i = nums.size()-1; i >= 0; i--)
        {
            for (int j = 0; j <= i; j++)
                curmax = nums[j] > nums[curmax] ? j : curmax;
            swap(nums[i], nums[curmax]);
            curmax = 0;
        }
        return nums;
    }
};
```

### **插入排序**
``` cpp
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        int curmax = 0;
        for (int i = 1; i < nums.size(); i++)
        {
            int tmp = nums[i];
            for (int j = i-1; j >=0 ; j--)
            {
                if (tmp < nums[j])
                    swap(nums[j],nums[j+1]);
                else
                    break;
            }
        }
        return nums;
    }
};
```
### **冒泡排序**
``` cpp
class Solution {
public:
    vector<int> sortArray(vector<int>& nums) {
        int curmax = 0;
        for (int i = 0; i < nums.size(); i++)//轮次，每轮找出一个最大值
        {
            for (int j = 0; j < nums.size() - 1 ; j++)
            {
                if (nums[j] > nums[j+1])
                    swap(nums[j],nums[j+1]);
            }
        }
        return nums;
    }
};
```
### **快速排序**
``` cpp
class Solution {
public:
    void qs(vector<int>& nums, int start, int end)
    {
        if (start >= end)
            return ;
        
        int privot_idx = rand()%(end-start + 1) + start;
        int privot = nums[privot_idx] ;
        swap(nums[privot_idx], nums[end]);
        int i = start, j = end - 1;
        while(i <= j)
        {
            while(i < end && nums[i] < privot) i++;
            while(j >= start && nums[j] > privot) j--;
            if (i <= j)
            {
               swap(nums[i], nums[j]); 
               i++; j--; //!!交换完一定要推进否则死锁
            }
        }
        swap(nums[i], nums[end]);
        qs(nums, start, i-1);
        qs(nums, i + 1, end);
    }
    vector<int> sortArray(vector<int>& nums) {
        srand(time(0));
        qs(nums,0,nums.size()-1);
        return nums;
    }
};
```

### 快排稳定版
```cpp
template<typename T>
void qsAsc(vector<T>& arr, int left, int right)
{
	if (left >= right)
		return;
	int privot_idx = (left + right) >> 1;
	T privot = arr[privot_idx];
	swap(arr[right], arr[privot_idx]);
	int i = left, j = right - 1;
	while (i <= j)
	{
		while (i <= j && arr[i] < privot) i++;
		while (i <= j && arr[j] > privot) j--;
		if (i <= j)
		{
			swap(arr[i], arr[j]);
			i++; j--;
		}
	}
	swap(arr[i], arr[right]);
	qsAsc(arr, left, i - 1);
	qsAsc(arr, i + 1, right);
}
```


### **堆排序(可优化)**
``` cpp
class Solution {
public:
    void adjust(vector<int>& nums, int root, int len)
    {
        int e = nums[root];
        for (int j = root*2; j <= len; j *= 2)
        {
            if (j < len && nums[j] < nums[j+1]) j++;
            if (e < nums[j]) swap(nums[j], nums[j/2]);
            else
                break;
        }
    }
    vector<int> sortArray(vector<int>& nums) {
        vector<int> sortnum(nums.size() + 1, 0);
        for (int i = 0; i < nums.size(); i++)
        {
            sortnum[i+1] = nums[i];
        }

        for (int i = nums.size()/2; i >= 1; i--)
            adjust(sortnum, i, nums.size());

        for (int i = nums.size()-1; i >= 1; i--)
        {
            swap(sortnum[i+1], sortnum[1]);
            adjust(sortnum, 1, i);
        }

        return {sortnum.begin() + 1, sortnum.end()};
    }
};
```

### **归并排序的递归和迭代**
``` cpp
class Solution {
public:
    void merge(vector<int>& nums, int left, int mid, int right)
    {
        vector<int> tmp(right-left +1);
        int i = left, j = mid + 1;
        int k = 0;
        while(i <= mid && j <= right)
        {
            tmp[k++] = nums[i] < nums[j] ? nums[i++] : nums[j++];
        }
        while(i <= mid) tmp[k++] = nums[i++];
        while(j <= right) tmp[k++] = nums[j++];

        for (int i = 0; i < tmp.size(); i++)
        {
            nums[i + left] = tmp[i];
        }
    }
    void mergesort_topdown(vector<int>& nums, int left, int right)
    {
        if (left >= right)
            return ;
        int mid = (left + right) >> 1;
        mergesort_topdown(nums,left, mid);
        mergesort_topdown(nums,mid+1, right);
        merge(nums,left,mid,right);
    }
    void mergesort_bottomup(vector<int>& nums, int left, int right)
    {
        if (left >= right)
            return ;
        int n = right-left + 1;
        for (int len = 1; len < n; len *= 2)
        {
            for(int i = left; i < left + n - len; i += 2*len)
            {
                int j = min(i + 2*len -1, left +n-1);
                int mid = i+len-1; //迭代法长度已经计算好，不需要(i+j)/2
                merge(nums,i,mid,j);
            }
        }
    }
    vector<int> sortArray(vector<int>& nums) {
        mergesort_bottomup(nums,0,nums.size()-1);
        return nums;
    }
};
```
## 非比较排序

### **计数排序**
**计数排序（Counting Sort）**是一种**非基于比较的排序算法**，适用于**值域较小**且**整数分布均匀**的情况。它的**时间复杂度是 O(n + k)**，其中 `n` 是元素个数，`k` 是值域范围。

---

 **📌 计数排序的具体步骤**
假设要排序的数组为 `arr`，其中所有元素均在 `[0, k]` 之间：

 **1️⃣ 统计每个元素的出现次数**
- 创建一个大小为 `k+1` 的计数数组 `count[]`，所有元素初始化为 `0`。
- 遍历 `arr`，对每个 `arr[i]`，增加 `count[arr[i]]` 的值。

 **2️⃣ 计算前缀和**
- `count[i]` 存储的是 **小于等于 `i` 的元素个数**，这样可以确定元素 `i` 在结果数组中的正确位置。

 **3️⃣ 根据 `count[]` 构造排序后的数组**
- 从原数组 **倒序遍历**，找到每个元素在 `count[]` 中的索引，并放入新数组 `output[]`，然后 `count[arr[i]]--`。

---

 **📌 代码实现**
 **🚀 C++ 版**
```cpp
#include <iostream>
#include <vector>

using namespace std;

void countingSort(vector<int>& arr) {
    if (arr.empty()) return;

    // 1️⃣ 找到最大值
    int maxVal = *max_element(arr.begin(), arr.end());

    // 2️⃣ 计数数组
    vector<int> count(maxVal + 1, 0);

    // 3️⃣ 统计每个元素出现的次数
    for (int num : arr) {
        count[num]++;
    }

    // 4️⃣ 计算前缀和，确定元素的最终位置
    for (int i = 1; i <= maxVal; i++) {
        count[i] += count[i - 1];
    }

    // 5️⃣ 构造排序结果
    vector<int> output(arr.size());
    for (int i = arr.size() - 1; i >= 0; i--) {
        output[count[arr[i]] - 1] = arr[i]; // 找到正确位置
        count[arr[i]]--; // 位置填充后，需要递减
    }

    // 6️⃣ 复制回原数组
    arr = output;
}

int main() {
    vector<int> arr = {4, 2, 2, 8, 3, 3, 1};
    countingSort(arr);

    cout << "排序后的数组: ";
    for (int num : arr) {
        cout << num << " ";
    }

    return 0;
}
```
---
 **📌 复杂度分析**
|  操作  |  复杂度  |
|  ----  |  ----  |
|  **遍历找到最大值**  |  `O(n)`  |
|  **创建并填充 `count[]`**  |  `O(n)`  |
|  **前缀和计算**  |  `O(k)`  |
|  **填充 `output[]`**  |  `O(n)`  |
|  **拷贝回 `arr[]`**  |  `O(n)`  |
|  **总复杂度**  |  `O(n + k)`  |

---
 **📌 适用场景**
✅ **适用：**
- 数据范围 `k` 不是很大（如 `k = O(n)`）。
- 数据是 **非负整数**（若包含负数，可偏移）。
- 适用于 **数据量大但值域小的情况**，如 **年龄、考试分数排序**。

❌ **不适用：**
- 如果 `k` 远远大于 `n`，则会浪费大量空间（如 `k = 10^9`，但 `n = 1000`）。
- 不能用于 **浮点数、字符串**（但可用于字符 ASCII 码排序）。
- 不能用于 **比较排序的场景**（如需要排序含负数的浮点数）。

---
 **📌 总结**
- **稳定排序** ✅ （相同元素相对顺序不会改变）
- **非比较排序** ✅ （不像快速排序、归并排序那样比较元素大小）
- **时间复杂度 O(n + k)** 🚀
- **空间复杂度 O(k)** 🛑（需要额外的 `count[]` ）
- **不适用于大范围值域的排序** ❌

---
如果你的数据范围较小且是整数，**计数排序是非常高效的选择！** 🚀

### **桶排序**
不使用排序的方法在O(n)时间复杂度找出相邻两个数的最大差值
在解决“最大相邻差值”问题时，这样计算桶参数的**根本目的是通过鸽巢原理将最大差值锁定在相邻桶之间**。具体原理如下：

---

#### **参数计算的三层逻辑**
1. **桶大小的设定逻辑**  
   ```cpp
   int bucket_size = max(1, (maxnum - minnum)/(n-1));
   ```
   - **数学保证**：当有n个元素时，至少存在一个空桶（n-1个桶时鸽巢原理成立）
   - **物理意义**：确保同一桶内元素的差值不可能成为全局最大值  
   - **极端处理**：`max(1, ...)`防止所有元素相同的情况

2. **桶数量的计算逻辑**  
   ```cpp
   int bucket_count = (maxnum - minnum)/bucket_size + 1;
   ```
   - **区间覆盖**：确保所有数值都能被分配到桶中  
   - **边界处理**：通过`+1`处理余数（如范围10，桶大小3时需4个桶）

3. **分桶策略的数学证明**  
   - **最大差值下限**：最小可能的最大差值为`bucket_size`  
   - **跨桶必然性**：任何大于`bucket_size`的差值必然跨越不同桶

---

``` cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

class Solution {
public:
    int maximumGap(vector<int>& nums) {
        const int n = nums.size();
        if (n < 2) return 0;
        
        int maxnum = *max_element(nums.begin(), nums.end());
        int minnum = *min_element(nums.begin(), nums.end());
        if (maxnum == minnum) return 0;
        
        // 计算桶参数,
        int bucket_size = max(1, (maxnum - minnum) / (n-1));
        int bucket_count = (maxnum - minnum) / bucket_size + 1;
        
        vector<pair<int, int>> buckets(bucket_count, {INT_MAX, INT_MIN});
        
        // 填充桶
        for (int num : nums) {
            int idx = (num - minnum) / bucket_size;
            buckets[idx].first = min(buckets[idx].first, num);
            buckets[idx].second = max(buckets[idx].second, num);
        }
        
        // 计算最大间距
        int max_gap = 0;
        int prev_max = buckets[0].second;
        for (int i = 1; i < bucket_count; ++i) {
            if (buckets[i].first == INT_MAX) continue; // 跳过空桶
            max_gap = max(max_gap, buckets[i].first - prev_max);
            prev_max = buckets[i].second;
        }
        
        return max_gap;
    }
};
```

### **基数排序**
#### **📌 什么是基数排序（Radix Sort）**
**基数排序（Radix Sort）** 是一种 **非比较排序**，适用于**整数排序**。它的核心思想是：**按照数位（从低位到高位）依次排序，最终得到有序数组**。

#### **📌 工作原理**
1. **确定数据的最大位数**（比如最大数是 5432，有 4 位）。
2. **按个位（最低位）排序**，所有数字按个位的大小放入桶中，然后按顺序取出。
3. **按十位排序**，同样按桶分类，再取出。
4. **按百位排序**，依次类推，直到最高位排序完成。
5. **最终数组是有序的**。

---
#### **📌 代码实现**
```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

// 获取最大值的位数
int getMaxDigits(vector<int>& nums) {
    int maxNum = *max_element(nums.begin(), nums.end());
    int digits = 0;
    while (maxNum > 0) {
        digits++;
        maxNum /= 10;
    }
    return digits;
}

// 基数排序
void radixSort(vector<int>& nums) {
    int maxDigits = getMaxDigits(nums);
    int exp = 1;  // 代表当前排序的位数（1, 10, 100...）

    for (int i = 0; i < maxDigits; i++) {
        vector<queue<int>> buckets(10);  // 10个桶（0-9）

        // 分配到桶
        for (int num : nums) {
            int digit = (num / exp) % 10;  // 取对应位的数
            buckets[digit].push(num);
        }

        // 按桶的顺序收集数据
        int index = 0;
        for (int j = 0; j < 10; j++) {
            while (!buckets[j].empty()) {
                nums[index++] = buckets[j].front();
                buckets[j].pop();
            }
        }

        exp *= 10;  // 处理下一个更高位
    }
}

int main() {
    vector<int> nums = {170, 45, 75, 90, 802, 24, 2, 66};
    radixSort(nums);
    for (int num : nums) {
        cout << num << " ";
    }
    return 0;
}
```

---
#### **📌 复杂度分析**
| 操作            | 复杂度 |
|---------------|-------|
| 计算最大位数   | \(O(n)\) |
| 每个位排序     | \(O(n)\) |
| 总共 \(d\) 位排序 | \(O(d \cdot n)\) |
| **总复杂度**    | **\(O(n \cdot d)\)** |

其中，\(d\) 是最大数字的位数，\(n\) 是数组大小。

---
#### **📌 适用场景**
##### **✅ 适用于：**
- **整数排序（非负数）**（如 **学号、手机号、身份证号、银行账号** 等）。
- **数据范围大，但位数小**（如 **最多 6~8 位的整数**）。
- **数据分布均匀**（如 **大数据排序、电话簿排序**）。

##### **❌ 不适用于：**
- **浮点数、负数排序**（需要特殊处理）。
- **非整数数据类型**（如字符串、结构体）。
- **数据值范围小但数量大的情况**（如 0~100 内 1 亿个数据，计数排序更优）。
- **内存占用敏感的情况**（需要额外存储桶）。

---
#### **📌 典型例题**
##### **📌 例题 1：LSD 基数排序**
**题目**：给定 **n** 个非负整数，使用 **基数排序** 对它们从小到大排序。

```cpp
Input: [170, 45, 75, 90, 802, 24, 2, 66]
Output: [2, 24, 45, 66, 75, 90, 170, 802]
```
🔹 **解法**：直接使用基数排序（代码见上）。

---

##### **📌 例题 2：最大数（Leetcode 179）**
🔹 **题目**：给定一组非负整数 `nums`，重新排列它们的顺序，使其组成的数字最大。例如：
```cpp
Input: [3,30,34,5,9]
Output: "9534330"
```
🔹 **解法**：
- 先将 `nums` 转换为字符串数组。
- **按自定义比较规则排序**，如果 `a + b > b + a`，则 `a` 应该在前面。
- 连接所有字符串得到结果。

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

string largestNumber(vector<int>& nums) {
    vector<string> strNums;
    for (int num : nums) {
        strNums.push_back(to_string(num));
    }
    
    // 自定义排序规则
    sort(strNums.begin(), strNums.end(), [](string& a, string& b) {
        return a + b > b + a;
    });

    // 拼接字符串
    string result = "";
    for (string& s : strNums) {
        result += s;
    }

    // 处理前导 0
    return result[0] == '0' ? "0" : result;
}

int main() {
    vector<int> nums = {3, 30, 34, 5, 9};
    cout << largestNumber(nums) << endl;  // Output: "9534330"
}
```

---
#### **📌 总结**
✅ **基数排序适用于整数排序，尤其适用于数据范围大但位数较小的情况。**  
✅ **时间复杂度 \(O(n \cdot d)\)，通常比比较排序（\(O(n \log n)\)）更快。**  
✅ **无法直接处理负数和浮点数，需要额外转换。**  
✅ **常用于大规模数据排序，如手机号、身份证号排序。** 🚀



# 二分搜索

# 栈
## 辅助栈
典型例题
1. 中缀表达式转后缀
2. 实现一个能O(1)找到最大最小值的栈

## 单调栈
### **单调栈的核心思路**
单调栈（Monotonic Stack）是一种特殊的栈数据结构，维护着栈内元素的**单调性**（递增或递减）。  
它常用于解决 **"下一个更大（小）元素"** 这类问题，并且能在 **O(n) 线性时间** 内完成。

---

### **🌟 核心思想**
1. **栈内元素保持单调（递增 / 递减）**
   - **单调递增栈**（栈底 -> 栈顶 递增）：用于找 **下一个更大元素**
   - **单调递减栈**（栈底 -> 栈顶 递减）：用于找 **下一个更小元素**

2. **入栈时，维护单调性**
   - 如果当前元素比栈顶元素大（或小），就**弹出栈顶**，因为栈顶元素已经找到了它的答案。

3. **适用于一类典型问题**
   - **找下一个/上一个更大（小）元素**
   - **柱状图最大矩形面积**
   - **滑动窗口最大值**
   - **温度 / 股票价格问题**

---

### **🔹 基本模板**
#### **1️⃣ 单调递增栈（找右侧第一个更大元素）**
```cpp
vector<int> nextGreaterElement(vector<int>& nums) {
    stack<int> stk;  // 单调递增栈（存索引或值）
    vector<int> res(nums.size(), -1); // 结果数组，默认 -1

    for (int i = 0; i < nums.size(); i++) {
        while (!stk.empty() && nums[i] > nums[stk.top()]) {
            res[stk.top()] = nums[i]; // 记录当前栈顶的 "下一个更大元素"
            stk.pop();  // 移除栈顶
        }
        stk.push(i);  // 压入当前元素索引
    }
    return res;
}
```
#### **🌟 解析**
- **栈内保持递增**，遇到 `nums[i]` **比栈顶大**，说明找到了栈顶的「下一个更大元素」，弹出栈顶。
- **时间复杂度**：每个元素最多进栈 & 出栈一次，**O(n)**。

---

#### **2️⃣ 单调递减栈（找右侧第一个更小元素）**
```cpp
vector<int> nextSmallerElement(vector<int>& nums) {
    stack<int> stk;
    vector<int> res(nums.size(), -1);

    for (int i = 0; i < nums.size(); i++) {
        while (!stk.empty() && nums[i] < nums[stk.top()]) {
            res[stk.top()] = nums[i]; 
            stk.pop();
        }
        stk.push(i);
    }
    return res;
}
```
- **栈内保持递减**，遇到更小元素就弹出栈顶。

---

### **📌 典型题目**
#### **1️⃣ 下一个更大元素（496. Next Greater Element I）**
**👉 题目**：给你一个数组，找到每个元素**右侧第一个比它大的元素**。如果不存在，返回 -1。

```cpp
vector<int> nextGreaterElement(vector<int>& nums) {
    stack<int> stk;
    vector<int> res(nums.size(), -1);

    for (int i = 0; i < nums.size(); i++) {
        while (!stk.empty() && nums[i] > nums[stk.top()]) {
            res[stk.top()] = nums[i];
            stk.pop();
        }
        stk.push(i);
    }
    return res;
}
```
**🔥 变种**
- **左侧第一个更大元素** → **从右往左遍历**
- **左侧第一个更小元素** → **从右往左遍历 + 递增栈**

---

#### **2️⃣ 柱状图最大矩形面积（84. Largest Rectangle in Histogram）**
- **题目**：给定柱状图，求最大矩形面积。
- **思路**：单调递增栈，**维护递增高度**，遇到更小的元素就计算面积。

```cpp
int largestRectangleArea(vector<int>& heights) {
    stack<int> stk;
    heights.push_back(0);  // 末尾加个 0，确保所有元素都能被处理
    int maxArea = 0;

    for (int i = 0; i < heights.size(); i++) {
        while (!stk.empty() && heights[i] < heights[stk.top()]) {
            int h = heights[stk.top()];
            stk.pop();
            int w = stk.empty() ? i : i - stk.top() - 1;
            maxArea = max(maxArea, h * w);
        }
        stk.push(i);
    }
    return maxArea;
}
```

---

#### **3️⃣ 滑动窗口最大值（239. Sliding Window Maximum）**
- **题目**：给定一个数组和窗口大小 `k`，找出**每个滑动窗口的最大值**。
- **思路**：单调**递减**双端队列（deque），维护窗口内最大值索引。

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;
    vector<int> res;

    for (int i = 0; i < nums.size(); i++) {
        if (!dq.empty() && dq.front() == i - k) 
            dq.pop_front(); // 移除过期的元素索引

        while (!dq.empty() && nums[dq.back()] < nums[i])
            dq.pop_back(); // 维护单调性（递减）

        dq.push_back(i);

        if (i >= k - 1) 
            res.push_back(nums[dq.front()]); // 记录最大值
    }
    return res;
}
```

---

### **🔹 总结**
|  问题类型  | 单调栈方向 | 作用 |
|:----:|:------:|:-----:|
| **找右侧第一个更大元素** | **递增栈** | 栈顶元素出栈时，当前元素是它的 "更大元素" |
| **找右侧第一个更小元素** | **递减栈** | 栈顶元素出栈时，当前元素是它的 "更小元素" |
| **柱状图最大矩形** | **递增栈** | 计算矩形宽度 |
| **滑动窗口最大值** | **递减队列** | 维护窗口内最大值 |

#### **💡 记住：**
- **递增栈**（找右侧更大元素）→ **小的弹出**
- **递减栈**（找右侧更小元素）→ **大的弹出**
- **O(n) 时间复杂度**，每个元素最多**进栈一次 & 出栈一次**。

单调栈是**维护区间信息**的神器，掌握后能高效解决很多数组 & 矩阵问题 🚀！



# 计算器——计算式值
## +-优先级大于*/

你的代码尝试实现一个计算器，**优先级设定为 `+`/`-` 高于 `*`/`/`**（和通常的数学规则相反）。思路挺有意思，但出现了多个 **逻辑和处理顺序上的问题**。下面我会从 **概念、代码问题、修改建议** 三部分详细说明。

---

### ✅ 1. 你的目标：非标准优先级计算

你希望：

- `+`、`-` **优先级高于** `*`、`/`
- 例子：
  - 表达式：`2 + 3 * 4`
  - 你希望：先算 `2 + 3 = 5`，再算 `5 * 4 = 20`
  - 而不是标准数学优先级那样：`3 * 4 = 12`，再 `2 + 12 = 14`

---

### ❌ 2. 代码中的问题

#### 🟠 问题1：逻辑不清晰导致某些操作数未入栈

你的分支中判断是否要 `nums.push_back(num)` 的时机非常混乱，特别在处理 `*` 和 `/` 时会出现 **漏 push 最后一个 num** 的情况。

---

#### 🟠 问题2：优先级处理混乱

你在处理 `+`/`-` 时尝试立即计算，而 `*`/`/` 时选择延后，但整个设计并没有正确分层优先级。例如下面这个判断：

```cpp
if (!ops.empty() && (ops.back() == '+' || ops.back() == '-'))
```

这个判断出现在多处，却没有明确表示你“已经将 `+`/`-` 优先计算”的意图，也没有分两个操作阶段。

---

#### 🟠 问题3：末尾数字没有正确处理

在循环完后，最后一个数字 `num` 可能没被处理。例如表达式 `1+2*3`，在遍历到末尾时 `3` 可能还没被加进 `nums` 里。

---

### ✅ 3. 正确思路与优化代码（+/- 优先级高）

最清晰的方法是 —— **分两个阶段：**

#### ✅ 阶段一：先处理 `+` 和 `-`（直接计算）

用一个中间数组记录：

- 遇到 `+` / `-`：立即计算结果并压入栈；
- 遇到 `*` / `/`：记录中间值，推迟处理。

#### ✅ 阶段二：再处理 `*` 和 `/`

你可以对上一步结果做普通乘除处理。

---

#### ✅ 改进后示例代码：

```cpp
#include <iostream>
#include <sstream>
#include <deque>
using namespace std;

int main() {
    string line;
    while (getline(cin, line)) {
        istringstream iss(line);
        deque<int> nums;
        deque<char> ops;

        int num;
        char op;
        iss >> num;
        nums.push_back(num);

        while (iss >> op >> num) {
            if (op == '+' || op == '-') {
                // 优先计算 + -（非标准优先级）
                int prev = nums.back(); nums.pop_back();
                nums.push_back(op == '+' ? prev + num : prev - num);
            } else {
                // * / 暂时保留
                nums.push_back(num);
                ops.push_back(op);
            }
        }

        // 第二阶段，处理剩下的 * /
        deque<int> finalNums;
        finalNums.push_back(nums.front());
        nums.pop_front();

        for (char op : ops) {
            int a = finalNums.back(); finalNums.pop_back();
            int b = nums.front(); nums.pop_front();
            if (op == '*') finalNums.push_back(a * b);
            else finalNums.push_back(a / b);
        }

        cout << finalNums.back() << endl;
    }
}
```

---

### 🧪 示例：

```txt
输入：2 + 3 * 4
按你逻辑：2 + 3 = 5，5 * 4 = 20
输出：20

输入：6 - 1 * 2
按你逻辑：6 - 1 = 5，5 * 2 = 10
输出：10
```

---

### ✅ 总结

| 问题 | 建议 |
|------|------|
| 表达式解析乱 | 改成逐步处理并分阶段更清晰 |
| 优先级处理错 | 用两个阶段来模拟不同优先级 |
| 代码结构复杂 | 用 `stringstream` 简化表达式读取 |

需要我再帮你把这个逻辑封装成函数形式或支持括号扩展吗？还是保留你原来的结构继续改？



# 堆
`priority_queue` 是一个常见的容器适配器，它提供了一个基于优先级的队列（最大堆或者最小堆）。在 C++ 中，`priority_queue` 是基于堆（heap）实现的，可以使用默认的最大堆或指定一个比较器来实现最小堆。

我将为你实现一个简单的 `priority_queue`，这个实现将使用最大堆（最大堆是默认行为）。

### **基本思路**

1. 使用数组或动态数组来存储数据。
2. 使用堆的插入和删除操作来维护队列的顺序。
3. 插入时，新的元素将被添加到数组的末尾，然后通过堆化（heapify）上浮操作保持堆的性质。
4. 删除最大元素时，交换堆顶元素和堆尾元素，然后通过堆化下沉操作保持堆的性质。

### **实现步骤**

- **插入操作**：插入元素后，通过上浮操作（bubble-up）将元素调整到合适的位置。
- **删除操作**：删除堆顶元素后，将堆尾元素移到堆顶，然后通过下沉操作（sink-down）将元素调整到合适的位置。
- **堆化操作**：用来维护堆的性质。

### **优先队列实现**

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>
using namespace std;

template<typename T>
class MyPriorityQueue {
private:
    vector<T> heap;

    // 向上堆化操作，保持最大堆性质
    void heapifyUp(int index) {
        while (index > 0) {
            int parent = (index - 1) / 2;
            if (heap[parent] < heap[index]) {
                swap(heap[parent], heap[index]);
                index = parent;
            } else {
                break;
            }
        }
    }

    // 向下堆化操作，保持最大堆性质
    void heapifyDown(int index) {
        int size = heap.size();
        while (2 * index + 1 < size) {
            int left = 2 * index + 1;
            int right = 2 * index + 2;
            int largest = index;

            if (left < size && heap[left] > heap[largest]) {
                largest = left;
            }
            if (right < size && heap[right] > heap[largest]) {
                largest = right;
            }
            if (largest != index) {
                swap(heap[index], heap[largest]);
                index = largest;
            } else {
                break;
            }
        }
    }

public:
    // 构造函数
    MyPriorityQueue() {}

    // 判断队列是否为空
    bool empty() const {
        return heap.empty();
    }

    // 返回队列中元素个数
    int size() const {
        return heap.size();
    }

    // 获取堆顶元素
    T top() const {
        if (empty()) {
            throw runtime_error("Priority Queue is empty");
        }
        return heap[0];
    }

    // 插入元素
    void push(const T& value) {
        heap.push_back(value);
        heapifyUp(heap.size() - 1);
    }

    // 删除堆顶元素
    void pop() {
        if (empty()) {
            throw runtime_error("Priority Queue is empty");
        }
        heap[0] = heap.back();  // 把堆尾元素移到堆顶
        heap.pop_back();         // 移除堆尾元素
        heapifyDown(0);          // 从堆顶开始堆化
    }
};

int main() {
    MyPriorityQueue<int> pq;

    // 插入元素
    pq.push(10);
    pq.push(20);
    pq.push(15);
    pq.push(30);
    pq.push(5);

    cout << "Priority Queue top: " << pq.top() << endl;  // 输出堆顶元素

    // 删除堆顶元素并输出堆顶
    pq.pop();
    cout << "Priority Queue top after pop: " << pq.top() << endl;

    pq.pop();
    cout << "Priority Queue top after another pop: " << pq.top() << endl;

    return 0;
}
```

### **解释：**
1. **`heapifyUp`**：当新元素插入到堆的末尾时，我们通过与父节点比较并交换来向上堆化，直到堆的性质得以恢复。
2. **`heapifyDown`**：当堆顶元素被移除后，我们把堆尾元素放到堆顶，然后从堆顶开始向下堆化，直到堆的性质得以恢复。
3. **`push`**：向队列插入新元素，插入后进行向上堆化。
4. **`pop`**：移除堆顶元素，并通过向下堆化来恢复堆的性质。
5. **`top`**：返回堆顶元素（即最大元素）。

### **主要操作时间复杂度：**
- **插入（push）**：O(log n)
- **删除（pop）**：O(log n)
- **获取堆顶元素（top）**：O(1)

### **输出示例：**
```
Priority Queue top: 30
Priority Queue top after pop: 20
Priority Queue top after another pop: 15
```

### **总结**
这就是一个自定义实现的最大堆优先队列。你可以在需要最大优先级元素的场景中使用这个类，或者根据需求修改它来实现最小堆（只需修改堆化过程中比较的方向）。


# 哈希
## 原地哈希
1. 找到最小没有出现的正整数或自然数

## 滚动哈希
**滚动哈希（Rolling Hash）** 是一种 **字符串哈希算法**，它允许在 **O(1) 时间** 计算字符串的哈希值，特别适用于 **字符串匹配和重复子串查找** 问题。

---
### **🚀 核心思想**
对于一个长度为 \( n \) 的字符串 \( S \)，如果我们需要计算其某个子串的哈希值，**传统的哈希计算方法需要遍历整个子串**，时间复杂度为 \( O(m) \)（其中 \( m \) 是子串长度）。

**滚动哈希的关键在于：**
1. **使用数学公式** 高效地从前一个子串的哈希值 **快速计算** 下一个子串的哈希值。
2. **避免重复计算**，将时间复杂度从 **\( O(m) \)** **降到 \( O(1) \)**。

---
### **📌 经典公式（基于 Rabin-Karp 算法）**
假设字符串是用 **26 进制**（只包含小写字母 'a' 到 'z'），我们用一个 **质数 \( p \)** 作为模数（例如 \( p = 10^9 + 7 \)）来防止哈希碰撞。

**子串哈希公式**：
\[
H(S[i:i+m]) = (H(S[i-1:i+m-1]) - S[i-1] \times P^{m-1}) \times P + S[i+m-1]
\]
其中：
- \( H(S[i:i+m]) \) 是子串 \( S[i:i+m] \) 的哈希值
- \( P \) 是进制（如 31）
- \( p \) 是取模的质数（如 \( 10^9 + 7 \)）
- \( S[i-1] \) 是 **移出的字符**
- \( S[i+m-1] \) 是 **加入的字符**
- 通过 **乘以 P 和取模**，让哈希值保持唯一性

---
### **🔥 代码示例：在字符串 `s` 中寻找长度为 `m` 的重复子串**
```cpp
#include <iostream>
#include <unordered_set>
using namespace std;

class Solution {
public:
    string longestDupSubstring(string s) {
        int left = 1, right = s.size();
        string res = "";
        while (left <= right) {
            int mid = (left + right) / 2;
            string dup = search(s, mid);
            if (!dup.empty()) {
                res = dup;
                left = mid + 1;  // 继续搜索更长的重复子串
            } else {
                right = mid - 1; // 长度过长，缩小搜索范围
            }
        }
        return res;
    }

    string search(const string& s, int len) {
        const int P = 31, MOD = 1e9 + 7;
        long long hash = 0, power = 1;
        unordered_set<long long> seen;

        // 计算初始哈希值
        for (int i = 0; i < len; i++) {
            hash = (hash * P + (s[i] - 'a' + 1)) % MOD;
            power = (power * P) % MOD;
        }
        seen.insert(hash);

        // 滚动计算哈希值
        for (int i = 1; i <= s.size() - len; i++) {
            hash = (hash * P - (s[i - 1] - 'a' + 1) * power % MOD + MOD) % MOD;
            hash = (hash + (s[i + len - 1] - 'a' + 1)) % MOD;
            if (seen.count(hash)) return s.substr(i, len);
            seen.insert(hash);
        }
        return "";
    }
};

int main() {
    Solution sol;
    cout << sol.longestDupSubstring("banana") << endl;  // 输出 "ana"
}
```
---
### **🎯 适用场景**
1. **字符串匹配**
   - **Rabin-Karp 算法** 使用滚动哈希来实现快速匹配。
   - **例子：查找字符串 `s` 是否包含 `t` 作为子串**。

2. **最长重复子串**
   - 通过 **二分查找 + 滚动哈希** 解决 **"最长重复子串"** 问题（如 Leetcode 1044）。

3. **DNA 序列检测**
   - **检测 DNA 序列是否包含重复模式**（如 `ACGTACGT`）。

4. **文本相似度检测**
   - **滚动哈希+哈希表** 可以在大文本中检测相似片段。

---
### **🚀 滚动哈希 vs 其他方法**
| **方法**           | **时间复杂度** | **优点**                                        | **缺点**                    |
|------------------|--------------|--------------------------------|----------------------------|
| **朴素暴力**       | \( O(nm) \)   | 简单易懂，适用于小数据量           | 计算每个子串哈希值，效率低 |
| **前缀哈希**       | \( O(n) \)    | 适用于**静态子串查询**             | **不能动态更新** |
| **KMP/Boyer-Moore** | \( O(n+m) \)  | 适用于**字符串匹配**               | 只适用于模式匹配问题 |
| **Rabin-Karp（滚动哈希）** | \( O(n) \)    | 适用于**重复子串/查找** | **可能有哈希碰撞** |

---
### **📌 总结**
🔹 **滚动哈希是一种高效的字符串处理算法，特别适用于字符串匹配、最长重复子串等问题。**  
🔹 **它的核心思想是利用数学公式，从前一个子串的哈希值快速计算下一个子串的哈希值，避免重复计算。**  
🔹 **与二分搜索结合，可以解决最长重复子串问题。** 🚀

# 树


## 普通树
## 二叉搜索树
## 路径问题
### 路径总和
找到和为target的路径，可以使用前缀和+哈希的方式
> 如果存在prefix = cur - target
> 说明有cur - prefix = target
> 即存在一条路径和为target，不要忽视前缀和为0时初始为1

### 最大路径和
### 最近公共祖先
### 序列化二叉树——(唯一表示——字符哈希)
同样可应用在异位字符串上，a2c3d6此种形式

# 拓扑排序
课程表问题
先建图，后搜索，dfs，bfs
拓扑排序的核心思想是 **依赖关系的顺序排序**，主要用于 **有向无环图（DAG，Directed Acyclic Graph）**。其核心概念如下：

---

## **🔹 核心思想**
1. **找入度为 0 的点（没有前置依赖的任务）**
   - 这些点可以最先执行。
   
2. **将这些点加入排序结果，并移除它们对其他点的影响**
   - 也就是更新相邻点的入度。
   
3. **重复上述过程，直到所有点都被处理**
   - 如果还有节点未处理，但没有入度为 0 的点，说明存在环（即任务存在循环依赖，无法完成）。

---

## **🔹 适用场景**
- **任务调度问题**（例如：课程安排 `Course Schedule`）
- **编译顺序问题**（例如：模块或文件的依赖管理）
- **数据流程依赖**（例如：计算表达式的求值顺序）

---

## **🔹 两种常见算法**
### **✅ 1. Kahn 算法（BFS，入度表法）**
基于 **入度** 的广度优先搜索：
1. 计算所有节点的 **入度**（被指向的次数）。
2. 将 **入度为 0** 的节点加入队列（可以安全执行）。
3. 从队列中取出节点，将其添加到拓扑排序结果中，并 **移除其对其他节点的影响**（减少相邻节点的入度）。
4. 如果某个节点的入度变为 0，则加入队列。
5. **重复直到队列为空**：
   - 如果最终排序结果中 **节点数 < 总节点数**，说明有环，拓扑排序失败。

```cpp
vector<int> topologicalSort(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> graph(numCourses);
    vector<int> inDegree(numCourses, 0);
    queue<int> q;
    vector<int> result;

    // 构建图和入度表
    for (auto& pre : prerequisites) {
        graph[pre[1]].push_back(pre[0]);
        inDegree[pre[0]]++;
    }

    // 将所有入度为 0 的节点加入队列
    for (int i = 0; i < numCourses; i++) {
        if (inDegree[i] == 0) {
            q.push(i);
        }
    }

    // BFS 进行拓扑排序
    while (!q.empty()) {
        int node = q.front();
        q.pop();
        result.push_back(node);
        
        for (int next : graph[node]) {
            inDegree[next]--;
            if (inDegree[next] == 0) {
                q.push(next);
            }
        }
    }

    return result.size() == numCourses ? result : vector<int>{}; // 若有环，返回空数组
}
```
**时间复杂度：** `O(N + M)`（`N` 是节点数，`M` 是边数）

---

### **✅ 2. DFS 拓扑排序**
基于 **递归 DFS** 的 **逆序入栈**：
1. 使用 **三色标记法** 进行 DFS：
   - `0`（白色）：未访问
   - `1`（灰色）：访问中（表示当前路径上，若遇到则有环）
   - `2`（黑色）：已完成
2. 递归访问所有未访问的节点：
   - 若遇到**灰色节点**，说明有环，无法拓扑排序。
   - 若遍历完一个节点，将其 **加入结果栈**（逆序存储）。
3. 最终输出 **栈的逆序**，即拓扑排序序列。

```cpp
class Solution {
public:
    vector<int> result;
    vector<int> visited;
    bool hasCycle = false;

    void dfs(int node, vector<vector<int>>& graph) {
        if (visited[node] == 1) { // 发现环
            hasCycle = true;
            return;
        }
        if (visited[node] == 2) return; // 已访问，跳过

        visited[node] = 1; // 访问中
        for (int next : graph[node]) {
            dfs(next, graph);
        }
        visited[node] = 2; // 访问完成
        result.push_back(node); // 逆序存储
    }

    vector<int> topologicalSort(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> graph(numCourses);
        visited.resize(numCourses, 0);
        
        for (auto& pre : prerequisites) {
            graph[pre[1]].push_back(pre[0]);
        }

        for (int i = 0; i < numCourses; i++) {
            if (visited[i] == 0) dfs(i, graph);
        }

        if (hasCycle) return {}; // 发现环，返回空数组
        reverse(result.begin(), result.end()); // 逆序
        return result;
    }
};
```
**时间复杂度：** `O(N + M)`

---

## **🔹 BFS vs. DFS 拓扑排序**
| 方法 | 适用情况 | 额外空间 | 是否能检测环 |
|------|--------|--------|------------|
| **Kahn (BFS) 算法** | 适合大规模数据流，能保证字典序最小的拓扑排序 | `O(N)`（队列 + 入度表） | ✅ 可以 |
| **DFS 逆序入栈** | 适合静态任务，递归方法直观 | `O(N)`（递归栈 + 结果数组） | ✅ 可以 |

---

## **🔹 例题**
1. **课程安排 I（Leetcode 207）**
   - 判断是否能完成课程（是否有环）。
2. **课程安排 II（Leetcode 210）**
   - 输出可行的学习顺序（拓扑排序）。
3. **任务调度（Leetcode 1203）**
   - 任务之间的优先顺序安排。
4. **编译顺序问题（LintCode 615）**
   - 依赖关系计算编译顺序。

---

## **🎯 结论**
- **拓扑排序用于 DAG**（有向无环图）。
- **Kahn 算法（BFS）适用于动态任务调度**。
- **DFS 逆序入栈适用于静态任务排序**。
- **如果有环，则不能进行拓扑排序**（意味着任务存在循环依赖）。

🔥 掌握这两种方法，AC 任何拓扑排序题目！


# 图论
## 解题方向
图论类问题种类繁多，但它们背后其实都有一些**通用的解题框架和套路**，掌握这些框架之后，面对大多数题型你都会有一个清晰的切入点。以下是一个比较系统的**图论题解题思路/模板框架**，我们按步骤整理出来：

---

### 🧭 图论类问题通用解题框架

---

#### ✅ 1. **识别图的类型**
这是**最重要的一步**，直接决定你后面用什么算法：

| 问题关键词 | 图的类型 |
|-------------|-----------|
| 无向图、双向通路 | 无向图 |
| 单向边、依赖关系、课程表 | 有向图 |
| 所有边权都为1 | 单位权图（可考虑 BFS） |
| 边有不同权值 | 加权图（用 Dijkstra/Bellman-Ford） |
| 有负权边 | Bellman-Ford、SPFA |
| DAG、无环图 | 有向无环图（拓扑排序） |
| 需要建边、转换规则 | 考虑是邻接表、邻接矩阵、边集 |

---

#### ✅ 2. **建图**
- 建图方式一般有 3 种：
  - **邻接表**（最常用，适合稀疏图）
  - **邻接矩阵**（适合稠密图/判断边是否存在）
  - **边集数组**（适合用 Kruskal、Bellman-Ford）

---

#### ✅ 3. **明确问题目标**
| 问题类型 | 解法思路 |
|----------|----------|
| 连通性、是否有环 | DFS/BFS |
| 最短路径 | BFS（单位权）/Dijkstra/Bellman-Ford |
| 拓扑排序（判断依赖顺序、检测环） | 拓扑排序 |
| 求路径数量、最长路径 | 动态规划（配合 DFS 或 拓扑） |
| 联通分量个数 | 并查集/DFS/BFS |
| 最小生成树 | Kruskal / Prim |
| 强连通分量 | Tarjan / Kosaraju |
| 割点、桥 | Tarjan |
| 染色问题 / 二分图 | BFS / DFS 染色 |
| 状态搜索 / 图隐式建图 | BFS（重点在状态设计） |

---

#### ✅ 4. **选择合适的算法框架**
| 算法 | 使用场景 |
|------|----------|
| DFS | 连通性、环检测、路径搜索、树形问题 |
| BFS | 最短路径、状态转换、层级遍历 |
| 拓扑排序 | DAG排序/环检测/任务调度 |
| Dijkstra | 单源最短路径（正权图） |
| Bellman-Ford | 存在负权边的图 |
| Kruskal / Prim | 最小生成树 |
| 并查集 | 连通分量、是否在一个集合内 |
| Tarjan | 强连通分量、割点、桥 |

---

#### ✅ 5. **细节处理**
- 是否有重边、自环？
- 是否可能不连通？（图是否为森林）
- 点和边是否从1开始/0开始？
- 多组输入、多测试用例？
- 数据范围是否需要用堆优化？（如 Dijkstra）

---

#### ✅ 6. **调试与验证**
- 打印中间变量、路径信息
- 验证特例（如图中只有一个节点，图为空等）

---

### 🎁 附赠：经典套路模板组合（关键词识别法）

| 关键词 | 建议做法 |
|--------|----------|
| 最短路径 + 边权为1 | BFS |
| 最短路径 + 边权不同 | Dijkstra |
| 是否有环 | DFS（递归栈） / 拓扑排序 |
| 是否连通 / 分量个数 | DFS / BFS / 并查集 |
| 所有点对最短路径 | Floyd-Warshall |
| 最大/最小生成树 | Kruskal / Prim |
| 能否分成两组 | 二分图染色 |
| 强连通 / 割点 / 桥 | Tarjan |
| 所有路径 / 特定路径 | DFS 回溯 |
| 状态变换问题 | 状态图 + BFS |

---


## dfs & bfs & 拓扑 适用场景
这是个很经典又重要的问题！在图论题中，DFS、BFS、拓扑排序虽然都是遍历图的方法，但适用场景有些不同。下面我帮你总结一下它们的典型使用场景和选择依据，方便你在刷题或做项目时判断用哪种：

---

### ✅ **DFS（深度优先搜索）**
#### 🧠 适合场景：
- 图的**连通性**判断（如判断是否有环、是否是一个连通图）
- **路径搜索**：找到从起点到终点的所有路径、最长路径
- 回溯 + 剪枝问题（常与 DFS 结合）
- **树形结构处理**（如树的遍历、树形DP）
- **求强连通分量**（Kosaraju 算法等）
- **拓扑排序**中的深搜版本

#### 📌 特点：
- 倾向于“走到底再回头”
- 用递归写法较直观（也可以用栈）

---

### ✅ **BFS（广度优先搜索）**
#### 🧠 适合场景：
- **最短路径**问题（单位权图中从起点到终点的最短路径）
- **最少步数**问题（迷宫、转化序列等）
- 多源 BFS（多个起点同时扩展）
- 状态搜索问题（例如“走迷宫”、“棋盘跳跃”）
- 最小操作步数的问题

#### 📌 特点：
- 一圈圈扩展，层层推进
- 通常用队列实现
- 可以很快找到目标（最短路径）

---

### ✅ **拓扑排序**
#### 🧠 适合场景：
- **有向无环图（DAG）上的依赖关系排序**
  - 任务调度（有前置依赖的任务）
  - 编译顺序问题（某些模块需在其他模块之后加载）
  - 检查是否存在环（拓扑排序失败即存在环）
- **动态规划 on DAG**（例如 DAG 上最长路径）

#### 📌 特点：
- 只能用于**有向图**
- 通常基于 DFS 或 BFS（入度为 0 入队）

---

### 🧭 如何选择：
| 问题特征 | 用法建议 |
|----------|----------|
| 求 **最短路径**（边权相同） | BFS |
| 求 **所有路径/某种路径** | DFS |
| 判断是否存在 **环**（有向/无向） | DFS |
| 有向图 + 有依赖顺序 | 拓扑排序 |
| 状态转换问题（如滑动拼图、数字变换） | BFS |
| 树上问题/递归搜索 | DFS |
| 多起点、最小步数问题 | BFS（多源 BFS） |
| DAG 上最长路径 | 拓扑排序 + DP |

---

## DFS
## BFS
## 拓扑


## 单源最短路径：Dijkstra
```cpp
vector<int> dis; //维护最短距离
dis[id] = 0;
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
pq.emplace(0,id);
while(!pq.empty())
{
    auto [time, start] = pq.top();
    pq.pop();
    if(time > dis[start])//旧
        continue;

    //从当前点start更新相邻点的最短dis，如果更新成功就加入队列
    for (start的相邻点point)
    {
        if(dis[point] > time+d){
            dis[point] = time + d;
            pq.emplace(time+d, point);
        }
    }   
}
```



## 全源最短路：Floyd 算法

求全源最短路劲
```cpp
for (k to n):
    for (i to n):
        if(dis[i][k] == INT_MAX)//i k不存在边
            continue;
        for(j to n):
            dis[i][j] = min(dis[i][j], dis[i][k]+dis[k][j]);
```

发生边更新x->y = cost,更新dis
```cpp
for(i to n):
    for (j to n):
        dis[i][j] = min(dis[i][j], dis[i][x]+cost+dis[y][j]);
```



## 最小生成树：prim & Kruskal 
以下是Prim算法和Kruskal算法的流程总结：

---

### **Prim算法流程**
1. **初始化**  
   • 将图中所有节点分为两个集合：**已访问集合S**和**未访问集合T**。初始时，S为空，T包含全部节点。
   • 随机选择一个起始节点（如节点0），将其加入S，并初始化距离数组`dis[]`：起始节点距离为0，其他节点距离为无穷大。

2. **迭代扩展**  
   • 在未访问集合T中，选择与已访问集合S距离最近的节点（即通过权重最小的边连接的节点），将其加入S。
   • 更新新加入节点的邻接节点在`dis[]`中的值：若存在更小的边权重，则更新为新的最小权重。

3. **终止条件**  
   • 重复上述步骤，直到所有节点均被加入S，形成包含`n-1`条边的最小生成树。若中途无法找到连接S与T的边，则图不连通，无解。

**核心思想**：始终维护一个连通子图，逐步扩展其边界，每次选择当前连通集到未连通节点的最短边。

---

### **Kruskal算法流程**
1. **边排序**  
   • 将所有边按权重从小到大排序。

2. **初始化并查集**  
   • 使用并查集（Disjoint Set）管理节点的连通性，初始时每个节点独立为一个集合。

3. **逐边选择**  
   • 按顺序遍历排序后的边，检查当前边的两个端点是否属于不同集合：
     ◦ **若否**：跳过该边（避免形成环）。
     ◦ **若是**：将边加入最小生成树，并通过并查集合并两个端点所在的集合。

4. **终止条件**  
   • 当生成树包含`n-1`条边时终止（n为节点数）。若遍历完所有边仍未满足条件，则图不连通。

**核心思想**：全局选择最小边，动态维护连通性，避免环路的形成。

---

### **算法对比**
| **特性**         | **Prim算法**                              | **Kruskal算法**                          |
|------------------|------------------------------------------|------------------------------------------|
| **适用场景**     | 稠密图（节点少、边多）                   | 稀疏图（边少、节点多）                   |
| **时间复杂度**   | O(V²)（邻接矩阵）或 O(E log V)（堆优化） | O(E log E)（边排序占主导）               |
| **核心操作**     | 维护节点到集合的距离                      | 排序边 + 并查集操作                      |
| **数据结构**     | 优先队列/堆、距离数组                    | 并查集、边列表                           |
| **是否需要连通图** | 必须连通                                 | 可处理非连通图（生成多个连通分量的森林） |

---

**总结**：  
• **Prim算法**以节点为中心，适合处理稠密图，通过贪心策略逐步扩展连通集。  
• **Kruskal算法**以边为中心，通过全局排序和并查集避免环路，适合边数较少的场景。两者的选择需结合具体图结构的稀疏性和实现复杂度。


# 并查集
并差集模板代码
```cpp
class UnionFind {
    vector<int> fa; // 代表元
    vector<int> sz; // 集合大小

public:
    int cc; // 连通块个数

    UnionFind(int n) : fa(n), sz(n, 1), cc(n) {
        // 一开始有 n 个集合 {0}, {1}, ..., {n-1}
        // 集合 i 的代表元是自己，大小为 1
        ranges::iota(fa, 0); // iota(fa.begin(), fa.end(), 0);
    }

    // 返回 x 所在集合的代表元
    // 同时做路径压缩，也就是把 x 所在集合中的所有元素的 fa 都改成代表元
    int find(int x) {
        // 如果 fa[x] == x，则表示 x 是代表元
        if (fa[x] != x) {
            fa[x] = find(fa[x]); // fa 改成代表元
        }
        return fa[x];
    }

    // 判断 x 和 y 是否在同一个集合
    bool is_same(int x, int y) {
        // 如果 x 的代表元和 y 的代表元相同，那么 x 和 y 就在同一个集合
        // 这就是代表元的作用：用来快速判断两个元素是否在同一个集合
        return find(x) == find(y);
    }

    // 把 from 所在集合合并到 to 所在集合中
    // 返回是否合并成功
    bool merge(int from, int to) {
        int x = find(from), y = find(to);
        if (x == y) { // from 和 to 在同一个集合，不做合并
            return false;
        }
        fa[x] = y; // 合并集合。修改后就可以认为 from 和 to 在同一个集合了
        sz[y] += sz[x]; // 更新集合大小（注意集合大小保存在代表元上）
        // 无需更新 sz[x]，因为我们不用 sz[x] 而是用 sz[find(x)] 获取集合大小，但 find(x) == y，我们不会再访问 sz[x]
        cc--; // 成功合并，连通块个数减一
        return true;
    }

    // 返回 x 所在集合的大小
    int get_size(int x) {
        return sz[find(x)]; // 集合大小保存在代表元上
    }
};

作者：灵茶山艾府
链接：https://leetcode.cn/discuss/post/mOr1u6/
```



# KMP

```cpp
class Solution {
public:
    vector<int> fail;
    void getFail(const string& needle)
    {
        fail.resize(needle.size(), 0);
        int j = 0;
        for (int i = 1; i < needle.size(); i++)
        {
            while(j > 0 && needle[j] != needle[i])
            {
                j = fail[j-1];
            }
            if (needle[i] == needle[j])
                j++;
            fail[i] = j;
        }
    }
    int strStr(string haystack, string needle) {
        getFail(needle);
        int i = 0, j = 0;
        for (; i < haystack.size(); i++)
        {
            while(j > 0 && needle[j] != haystack[i])
                j = fail[j-1];
            if (haystack[i] == needle[j])
                j++;
            if(j == needle.size())
                return i - needle.size() + 1;
        }
        return -1;
    }
};
```



# 最长递增 & 非递减 子序列