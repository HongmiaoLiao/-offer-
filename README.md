# (剑指offer 专项突击版)

[leetcode页面](https://leetcode.cn/problem-list/e8X3pBZi/)

[toc]

## 整数

知识点：

* 1 快速乘：类似[快速幂](https://leetcode.cn/problems/powx-n/),在快速幂算法的基础上，将乘法运算改成加法运算即可。
* 2 字符对应的ASCII码，一般不用具体知道，映射时可以(char - 'a')这样用

### 001.整数除法

注意点：复杂的边界条件优先处理，使除数和被除数先变为负数，这样不会有溢出问题。

方法：
计算X/Y = Z, 求Z \* Y >= X > (Z+1) \* Y
用二分查找找到最大的Z，用快速乘算法计算Z * Y的值

C++代码：

```C++
class Solution {
public:
    // 堪比困难题的简单题
    int divide(int a, int b) {
        // 考虑被除数为最小值的情况
        if (a == INT_MIN) {
            if (b == 1) {
                return INT_MIN;
            }
            if (b == -1) {
                return INT_MAX;
            }
        }
        // 考虑出数为最小值的情况
        if (b == INT_MIN) {
            return a == INT_MIN ? 1: 0;
        }
        // 考虑被除数为0的情况
        if (a == 0) {
            return 0;
        }
        // 将所有的正整数取相反数
        bool rev = false;
        if (a > 0) {
            a = -a;
            rev = !rev;
        }
        if (b > 0) {
            b = -b;
            rev = !rev;
        }

        int left = 1;
        int right = INT_MAX;
        int res = 0;
        while (left <= right) {
            int mid = left + ((right - left) >> 1);
            if (quickAdd(a, b, mid)) {
                res = mid;
                if (mid == INT_MAX) {
                    break;
                }
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return rev? -res: res;
    }
    // 快速乘
    // c * b < a返回false， // 注意a, b是负数，注意负数的大小比较
    bool quickAdd(int a, int b, int c) {
        int res = 0;    // c * b;
        int add = b;
        while (c) {
            if (c & 1) {
                // 需要保证result + add >= x
                if (res < a - add) {
                    return false;
                }
                res += add;
            }
            if (c != 1) {
                if (add < a - add) { // 相当于提前停止，不需要加完后再比较并返回。
                    return false;
                }
                add += add;
            }
            c >>= 1;
        }
        return true;
    }
};
```

### 002.二进制求和

注意点：字符串的起始顺序和二进制数的顺序相反。

方法：模拟
先把两个字符串翻转对齐，这样从下标0开始遍历就是从最低位开始遍历，再将对应每一位逐个相加，最后将长度较长的部分加上。

C++代码：

```C++
class Solution {
    string addBinary(string a, string b) {
        string res;
        // 翻转两个字符串，遍历顺序从最低位开始
        reverse(a.begin(), a.end());
        reverse(b.begin(), b.end());
        // 遍历到最长的字符串长度
        int n = max(a.size(), b.size());
        int carry = 0;  // 记录进位
        for (int i = 0; i < n; ++i) {
            // 下标超过长度部分直接加0，这样使代码简洁，行为一致
            carry += i < a.size() ? (a[i] == '1') : 0;
            carry += i < b.size() ? (b[i] == '1') : 0;
            // carry的值可能为0，1，2，3
            res += carry % 2 ? "1" : "0";
            carry /= 2; // 作为进位，也就是下一次相加的值。
        }
        // 最后如果还有进位
        if (carry) {
            res += "1";
        }
        // 反转回答案
        reverse(res.begin(), res.end());
        return res;
    }
}

```

Java代码：

```Java
class Solution {
    public String addBinary(String a, String b) {
        // 用StringBuffer做字符串操作，效率较高
        StringBuffer res = new StringBuffer();
        // 遍历到最长的字符串长度
        int n = Math.max(a.length(), b.length());
        int carry = 0;  // 记录进位
        for (int i = 0; i < n; ++i) {
            // 下标超过长度部分直接加0，这样使代码简洁，行为一致
            // 注意从低位开始，也就是字符串的后面开始计算
            carry += i < a.length() ? (a.charAt(a.length()-1-i) - '0') : 0;
            carry += i < b.length() ? (b.charAt(b.length()-1-i) - '0') : 0;
            // carry的值可能为0，1，2，3
            res.append((char)(carry % 2 + '0'));
            carry >>= 1; // 作为进位，也就是下一次相加的值。
        }
        // 最后如果还有进位
        if (carry > 0) {
            res.append('1');
        }
        // 反转回答案
        res.reverse();
        return res.toString();
    }
}
```

### 003.前n个数字二进制中1的个数

对于奇数：二进制表示中，奇数一定比前面那个偶数多一个 1，因为多的就是最低位的 1。
对于偶数：二进制表示中，偶数中 1 的个数一定和除以 2 之后的那个数一样多。因为最低位是 0，除以 2 就是右移一位，也就是把那个 0 抹掉而已，所以 1 的个数是不变的。

[方法](https://leetcode.cn/problems/counting-bits/solution/hen-qing-xi-de-si-lu-by-duadua/)：
动态规划:
$$
f(i) =
\begin{cases}
    f(i-1) + 1, &奇数\\
    f(i/2), 偶数
\end{cases}
$$

C++代码：

```C++
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> res(n+1);
        res[0] = 0;
        for (int i = 1; i <= n; ++i)
            if (i % 2 == 0)
                res[i] = res[i / 2];    // 偶数
            else
                res[i] = res[i-1] + 1;  // 奇数
                
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int[] countBits(int n) {
        int[] res = new int[n+1];
        res[0] = 0;
        for (int i = 1; i <= n; ++i) {
            if (i % 2 == 0) {
                res[i] = res[i / 2];    // 偶数
            } else {
                res[i] = res[i-1] + 1;  // 奇数
            }
        }
        return res;
    }
}
```

### 004 只出现一次的数字

方法：[位数统计](https://leetcode.cn/problems/WGki4K/solution/by-ac_oier-npwu/)
使用一个长度为32的数组记录下所有数值的每一位共出现了多少次1，再对数组的每一位进行mod 3操作，重新拼凑出只出现一次的数值

C++代码：

```C++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        // 对于每一位，遍历一次所有数字进行统计1的个数
        for (int i = 0; i < 32; ++i) {
            int cnt1 = 0;// 统计每一位1的个数
            for (int num: nums) {
                // 右移i位再与1, 就是判断那一位是否为1
                cnt1 += (num >> i) & 1;
            }
            // cnt只可能为1或3，如果为1，则右移i位在相加
            // 等价于将res的第i位置为1
            if (cnt1 % 3) {
                res |= 1 << i;
            }
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        // 对于每一位，遍历一次所有数字进行统计1的个数
        for (int i = 0; i < 32; ++i) {
            int cnt = 0;    // 统计每一位1的个数
            for (int num: nums) {
                // 右移i位再与1, 就是判断那一位是否为1
                cnt += (num >> i) & 1;  
            }
            // cnt只可能为1或3，如果为1，则右移i位在相加
            // 等价于将res的第i位置为1
            res += (cnt % 3) << i;  
        }
        return res;
    }
}
```

### 005 单词长度的最大乘积

方法：由于只包含小写字母，于是用一个int类型的值的26位表示字符串中的字母是否出现过，如果代表两个字符串的int类型的值按位与的结果为0，代表这两个字符串没有重复的字母。先记下所有字符串的int值，然后两两比较求得结果。
优化点：不需要记下所有字符串，比如met和meet，记下重复字符较多，即长度较长的那个字符串长度即可。

C++代码：

```C++
class Solution {
public:
    int maxProduct(vector<string>& words) {
        // key：字符串的int值， value：对应的最长字符串长度
        unordered_map<int,int> map;
        int length = words.size();
        for (int i = 0; i < length; i++) {
            int mask = 0;   // 每个字符串对应int值
            string word = words[i];
            int wordLength = word.size();
            for (int j = 0; j < wordLength; j++) {
                // 对字符对应的位置为1，用|或 操作可避免重复置位
                mask |= 1 << (word[j] - 'a');
            }
            // 如果长度比相同int值的长，则更新map中记录的字符串长度值
            if(map.count(mask)) {
                if (wordLength > map[mask]) {
                    map[mask] = wordLength;
                }
            } else {    // 原来没有记录，直接插入
                map[mask] = wordLength;
            }
            
        }
        int maxProd = 0;// 最大相乘的答案值
        // 对记录的map每个值两两遍历计算
        for (auto [mask1, _] : map) {
            int wordLength1 = map[mask1];
            for (auto [mask2, _] : map) {
                // 两个int字符串对应int值按位与为0，说明没有重复字符，进行计算
                if ((mask1 & mask2) == 0) {// 注意&优先级小于==
                    int wordLength2 = map[mask2];
                    maxProd = max(maxProd, wordLength1 * wordLength2);
                }
            }
        }
        return maxProd;
    }
};
```

Java代码：

```Java
class Solution {
    public int maxProduct(String[] words) {
        // key：字符串的int值， value：对应的最长字符串长度
        Map<Integer, Integer> map = new HashMap<>();
        int n = words.length;
        for (int i = 0; i < n; ++i) {
            int mask = 0;   // 每个字符串对应int值
            String word = words[i];
            int wordLength = word.length();
            for (int j = 0; j < wordLength; ++j) {
                // 对字符对应的位置为1，用|或 操作可避免重复置位
                mask |= 1 << (word.charAt(j) - 'a');
            }
            // 如果长度比相同int值的长，则更新map中记录的字符串长度值
            if (map.containsKey(mask)) {
                if (wordLength > map.get(mask)) {
                    map.put(mask, wordLength);
                }

            } else {// 原来没有记录，直接插入
                map.put(mask, wordLength);
            }
        }
        int res = 0; // 最大相乘的答案值
        Set<Integer> maskSet = map.keySet(); // 用于遍历的每个键
        // 对记录的map每个值两两遍历计算
        for (int mask1: maskSet) {
            int wordLength1 = map.get(mask1);
            for (int mask2: maskSet) {
                // 两个int字符串对应int值按位与为0，说明没有重复字符，进行计算
                if ((mask1 & mask2) == 0) {// 注意&优先级小于==
                    int wordLength2 = map.get(mask2);
                    res = Math.max(res, wordLength1 * wordLength2);
                }
            }
        }
        return res;
    }
}
```

### 006 排序数组中两个数字之和

方法：双指针，一个从左边，另一个从右边，两数和偏大时右边左移，两数和偏小是，左边右移。因为数组是有序的，且仅存在一个有效答案，所以不会错过答案。

C++代码：

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int left = 0;
        int right = numbers.size() - 1;
        vector<int> res(2); // 答案仅两个数
        while (left < right) {
            int sum = numbers[left] + numbers[right];
            if (sum == target) {    // 找到答案，直接返回
                res[0] = left;
                res[1] = right;
                return res;
            } else if (sum < target) {  // 偏小，左指针右移
                ++left;
            } else {    // 偏大，右指针左移
                --right;
            }
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0;
        int right = numbers.length - 1;
        int[] res = new int[2]; // 答案仅两个数
        while (left < right) {
            int sum = numbers[left] + numbers[right];
            if (sum == target) { // 找到答案，直接返回
                res[0] = left;
                res[1] = right;
                return res;
            } else if (sum < target) {// 偏小，左指针右移
                ++left;
            } else {// 偏大，右指针左移
                --right;
            }
        }
        return res;
    }
}
```

## 数组

数组问题常用方法（同样适用于字符串，类似于字符数组）：

* 双指针，排序数组的左右两边向内靠拢，或者快慢指针
* 滑动窗口
* 二分查找（后面专门章节）
* 前缀和（本专题重点）：一般用数组或哈希表存储前缀和信息，数组中两个位置的前缀和相减，就能的到中间窗口的总量信息。（前缀和不一定是数字之和，也可能是计数信息，数组也不一定是一维，需要注意变通）。如果数组全为正数，前缀和具有有序递增特性

滑动窗口格式

```C++
int left = 0;
int right = 0;
while (right < n) {
    ...nums[right];
    while (left < right && ...) {
        ...
        ...nums[left];
        ++left;
    }
    ...
    ++right;
}
```

前缀和格式

```C++
int preSum = 0;
for (int num: nums) {
    preSum += num
    if (map.count(preSum)) {
        ...
    } else {
        ...
    }
    map[preSum] = ...;
}

```

### 007 数组中和为0的三个数

方法：本题可以看做是第六题的扩展，先对数组进行排序，使用三个指针i，j，k代表三个要找的数的下标，枚举i确定第一个数，j、k分别从右边剩下的部分数组进行搜索，j = i + 1， k = n - 1，j、k两边向内靠拢

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> res; 
        sort(nums.begin(), nums.end()); // 排序数组
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            // 跳过i重复的数，避免重复枚举，如果有重复，每次i都是取重复的第一个数
            if (i > 0 && nums[i] == nums[i-1]) {
                continue;
            }
            // 剩下右边部分数组的两边
            int j = i + 1;
            int k = n - 1;
            while (j < k) {
                // 同样，跳过j重复的数。
                while (j > i + 1 && j < n && nums[j] == nums[j-1]) {
                    ++j;    // 去除重复
                }
                // 去重时下标超过了j，跳出循环，注意别漏
                if (j >= k) {
                    break;
                }
                // 三数之和
                int sum = nums[i] + nums[j] + nums[k];
                // 由于剩下右边部分的数组和为0的值可能不只一个，放入答案后不会跳出循环
                //  同第六题部分 
                if (sum == 0) {
                    res.push_back({nums[i], nums[j], nums[k]});
                    ++j;
                } else if (sum > 0) {
                    --k;
                } else {
                    ++j;
                }
            }
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        Arrays.sort(nums);// 排序数组
        int n = nums.length;
        for (int i = 0; i < n; ++i) {
            // 跳过i重复的数，避免重复枚举，如果有重复，每次i都是取重复的第一个数
            if (i > 0 && nums[i] == nums[i-1]) {
                continue;
            }
            // 剩下右边部分数组的两边
            int j = i + 1;
            int k = n - 1;
            while (j < k) {
                // 同样，跳过j重复的数。
                while (j > i + 1 && j < n && nums[j] == nums[j-1]) {
                    ++j;    // 去除重复
                }
                // 去重时下标超过了j，跳出循环，注意别漏
                if (j >= k) {
                    break;
                }
                // 三数之和
                int sum = nums[i] + nums[j] + nums[k];
                // 由于剩下右边部分的数组和为0的值可能不只一个，放入答案后不会跳出循环
                //  同第六题部分 
                if (sum == 0) {
                    res.add(Arrays.asList(nums[i], nums[j], nums[k]));
                    ++j;
                } else if (sum > 0) {
                    --k;
                } else {
                    ++j;
                }
            }
        }
        return res;
    }
}
```

### 008 和大于等于target的最短子数组

方法：可用前缀和+二分查找，即先计算出每个位置前缀和作为数组，再对每个枚举区间用二分查找，找到满足的条件的区间。
本体的最优解法为滑动窗口，窗口右边先增大到窗口内的和大于等于目标，然后窗口左边收缩到窗口内的和小于目标，在收缩过程总计算窗口大小，也即子数组长度。

C++代码：

```C++
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int res = INT_MAX;  // 答案初始化为最大int
        int left = 0;   // 窗口左边
        int right = 0;  // 窗口右边
        int n = nums.size();
        int sum = 0;    // 窗口内的元素之和
        while (right < n) {
            // 窗口右边扩大
            sum += nums[right];
            // 窗口左边收缩
            while (sum >= target) {
                // 收缩过程记下最小窗口大小，即最短子数组
                res = min(res, right - left + 1);
                sum -= nums[left];
                ++left;
            }
            ++right;
        }
        // 如果答案还未最大int值，则不存在符合条件数组，返回0
        return res == INT_MAX? 0: res;
    }
};
```

Java代码：

```Java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int n = nums.length;
        int left = 0;   // 窗口左边
        int right = 0;  // 窗口右边
        int sum = 0;    // 窗口内的元素之和
        int res = Integer.MAX_VALUE; // 答案初始化为最大int
        while (right < n) {
            // 窗口右边扩大
            sum += nums[right];
            // 窗口左边收缩
            while (sum >= target) {
                // 收缩过程记下最小窗口大小，即最短子数组
                res = Math.min(res, right - left + 1);
                sum -= nums[left];
                ++left;
            }
            ++right;
        }
        // 如果答案还未最大int值，则不存在符合条件数组，返回0
        return res == Integer.MAX_VALUE? 0: res;
    }
}
```

### 009 乘积小于k的子数组

方法：滑动窗口，先窗口右边扩大到窗口内元素乘积大于k，然后收缩窗口左边，知道窗口内元素乘积小于k，需要注意的是，每次右指针移动到一个新的位置，答案应该加上窗口大小那么多种的组合数量。
$nums[right]$
$nums[right-1], nums[right]$
$nums[right-2], nums[right-1], nums[right]$
$nums[left]，  nums[left+1], ......, nums[right-1], nums[right]$
共有 right - left + 1 种，这样每次窗口移动能够穷举到所有可能的子数组个数

C++代码：

```C++
class Solution {
public:
    int numSubarrayProductLessThanK(vector<int>& nums, int k) {
        int res = 0;
        int n = nums.size();
        int mul = 1;    // 窗口内元素乘积
        int left = 0;    // 窗口左边
        int right = 0;  // 窗口右边
        while (right < n) {
            // 窗口右边扩大
            mul *= nums[right];
            // 窗口左边收缩
            while (left <= right && mul >= k) {
                mul /= nums[left];
                ++left;
            }
            //每次右指针位移到一个新位置，应该加上窗口大小种数组组合：
            res += right - left + 1;
            ++right;
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int numSubarrayProductLessThanK(int[] nums, int k) {
        int res = 0;
        int n = nums.length;
        int mul = 1;    // 窗口内元素乘积
        int left = 0;    // 窗口左边
        int right = 0;  // 窗口右边
        while (right < n) {
            // 窗口右边扩大
            mul *= nums[right];
            // 窗口左边收缩
            while (left <= right && mul >= k) {
                mul /= nums[left];
                ++left;
            }
            //每次右指针位移到一个新位置，应该加上窗口大小种数组组合：
            res += right - left + 1;
            ++right;
        }
        return res;
    }
}
```

### 010 和为k的子数组

方法：使用一个HashMap存放前缀和以及此前缀和出现的次数，然后在遍历中判断是否存在（此时的前缀和-k）== 之前出现过的前缀和，如果有，说明此窗口（子数组）内的和为k，结果加上那个前缀和出现的次数。

注意此题中，由于要求结果是等于，而且nums有正有负，和为k的子数组一个右边界对应不只一个左边界，所以不可以用滑动窗口

C++代码：

```C++
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int res = 0;
        // key：前缀和， value：该前缀和出现的次数
        unordered_map<int, int> mp;
        // 前缀和为0的有一个，为了一个数就刚好为k可以统计进结果
        mp[0] = 1;  
        int preSum = 0; // 前缀和
        for (int& num: nums) {
            preSum += num;
            // 查找之前符合条件的的前缀和
            if (mp.find(preSum - k) != mp.end()) {
                res += mp[preSum-k];
            }
            // 加上遍历到此处的前缀和次数
            ++mp[preSum];
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int res = 0;
        // key：前缀和， value：该前缀和出现的次数
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        int preSum = 0; // 前缀和
        for (int num: nums) {
            preSum += num;
            // 查找之前符合条件的的前缀和
            if (map.containsKey(preSum - k)) {
                res += map.get(preSum - k);
            }
            // 加上遍历到此处的前缀和次数
            map.put(preSum, map.getOrDefault(preSum, 0) + 1);
        }
        return res;
    }
}
```

### 011 0和1个数相同的子数组

方法：将0当作-1，使用前缀和方法，在遍历数组过程中用HashMap存储前缀和以及该前缀和对应的数组下标，如果遍历到的当前的前缀和与前面的某个前缀和相等，则说明两个前缀和之间的区间和为0，于是记下区间长度。

C++代码：

```C++
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        int res = 0;
        int n = nums.size();
        // key：前缀和，value：对应数组下标
        unordered_map<int, int> mp; 
        mp[0] = -1; // 前缀和为0的下标为-1
        int preSum = 0; // 前缀和
        for (int i = 0; i < n; ++i) {
            if (nums[i] == 1) {
                ++preSum;
            } else {// 如果为0，则当作-1计算
                --preSum;
            }
            // 如果前面出现过相同前缀和，说明这个区间内前缀和为0
            if (mp.find(preSum) != mp.end()) {
                res = max(res, i - mp[preSum]);
            } else {// 否则记录前缀和
                // 对于相同前缀和，只记下了最靠前的下标
                mp[preSum] = i;
            } 
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int findMaxLength(int[] nums) {
        int res = 0;
        int n = nums.length;
        // key：前缀和，value：对应数组下标
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, -1); // 前缀和为0的下标为-1
        int preSum = 0; // 前缀和
        for (int i = 0; i < n; ++i) {
            if (nums[i] == 1) {
                ++preSum;
            } else {// 如果为0，则当作-1计算
                --preSum;
            }
            // 如果前面出现过相同前缀和，说明这个区间内前缀和为0
            if (map.containsKey(preSum)) {
                res = Math.max(res, i - map.get(preSum));
            } else {// 否则记录前缀和
                // 对于相同前缀和，只记下了最靠前的下标
                map.put(preSum, i);
            }
        }
        return res;
    }
}
```

### 012 左右两边子数组的和相等

方法：先计算数组所有元素的和，然后使用前缀和，如果（所有元素的和-中心轴元素-中心轴元素前一个的前缀和）== 中心轴元素前一个的前缀和，说明中心轴元素左右两边的和相等。

C++代码

```C++
class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        int sum = 0;
        // 计算所有元素的和
        for (int num: nums) {
            sum += num;
        }
        int preSum = 0; // 前缀和
        int res = -1;   // 如果不存在，返回-1
        for (int i = 0; i < nums.size(); ++i) {
            // 根据中心轴元素判断两边是否相等
            if (sum - nums[i] - preSum == preSum) {
                res = i;
                break;
            }
            // 前缀和加上中心轴元素要放在判断后面，这样才分得出轴前一个的前缀和
            preSum += nums[i];
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int pivotIndex(int[] nums) {
        int sum = 0;
        // 计算所有元素的和
        for (int num: nums) {
            sum += num;
        }
        int preSum = 0; // 前缀和
        for (int i = 0; i < nums.length; ++i) {
            // 根据中心轴元素判断两边是否相等
            if (sum - nums[i] - preSum == preSum) {
                return i;
            }
            // 前缀和加上中心轴元素要放在判断后面，这样才分得出轴前一个的前缀和
            preSum += nums[i];
        }
        return -1;
    }
}
```

### 013 二维子矩阵的和

方法：计算矩阵每一个元素(x,y)的左上角(0, 0)开始到(x, y)的前缀和存放在一个同等大小。矩阵前缀和的计算为一个格子上面的前缀和加上格子左边的前缀和减去重复相加的部分再加上该格子的值。之后计算矩形区域的和就可以用 (右下角前缀和 - 左下角前缀和 - 右上角前缀和 + 左上角前缀和)之间计算出来，加上左上角前缀和
相当于加回多减掉的重复部分。

tips：前缀和矩阵大一圈方便边界的计算，可以把前缀和下标看成是格子的线的右下较位置。

C++代码：

```C++
class NumMatrix {
public:
    vector<vector<int>> v;  // 前缀和矩阵
    NumMatrix(vector<vector<int>>& matrix) {
        int m = matrix.size();
        int n = matrix[0].size();
        // 矩阵维度大一圈，计算前缀和时方便边界的计算
        v.resize(m+1, vector<int>(n+1));
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                // 矩阵前缀和的计算为一个格子上面的前缀和加上格子左边的前缀和减去重复相加的部分再加上该格子的值
                v[i+1][j+1] = v[i][j+1] + v[i+1][j]
                             + matrix[i][j] - v[i][j];
            }
        }
    }
    
    int sumRegion(int row1, int col1, int row2, int col2) {
        // 右下角前缀和 - 左下角前缀和 - 右上角前缀和 + 左上角前缀和
        return v[row2+1][col2+1] - v[row2+1][col1] - v[row1][col2+1]
                + v[row1][col1];
    }
};
```

Java代码：

```Java:
class NumMatrix {
    
    int[][] preSumMatrix;
    public NumMatrix(int[][] matrix) {
        int m = matrix.length;
        int n = matrix[0].length;
        // 矩阵维度大一圈，计算前缀和时方便边界的计算
        preSumMatrix = new int[m+1][n+1];
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                // 矩阵前缀和的计算为一个格子上面的前缀和加上格子左边的前缀和减去重复相加的部分再加上该格子的值
                preSumMatrix[i][j] = preSumMatrix[i-1][j] + preSumMatrix[i][j-1]
                                 - preSumMatrix[i-1][j-1] + matrix[i-1][j-1];
            }
        }
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
        // 右下角前缀和 - 左下角前缀和 - 右上角前缀和 + 左上角前缀和
        return preSumMatrix[row2+1][col2+1] - preSumMatrix[row2+1][col1]
               - preSumMatrix[row1][col2+1] + preSumMatrix[row1][col1];
    }
}
```

## 字符串

字符串解法大多类似数组，使用双指针、滑动窗口，但一般也会更多用到HashMap（或者字符到数组映射）。
字符串的滑动窗口格式

```C++

```

### 014 字符串中的变位词
