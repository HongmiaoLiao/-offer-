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
字符串的滑动窗口（一般会是两个字符串的比较）格式

```C++
void slidingWindow(string s1, string s2) {
    unordered_map<char, int> windows;   // 窗口内字符
    unordered_map<char, int> need;  // 题目需求条件的记录
    for (char c: s2) {
        ++need[c];
    }
    int left = 0;
    int right = 0;
    int valid = 0;
    while (right < s1.size()) {
        char c = s1[right]; // c1是移入窗口的字符
        ++right;    // 右移窗口
        ...
        while (left < right && ...) {// 收缩条件
            char c2 = s[left];  // c2是移出窗口的字符
            ++left;
            ...
        }
    }
}
```

### 014 字符串中的变位词

方法：利用滑动窗口，先用HashMap记下s1（子串）需要的字符及其个数，然后，用滑动窗口遍历s2，右窗口扩大时，需要的字符数减少，左窗口收缩时，需要的字符串增加。当某个字符数减少到小于0，则左窗口收缩。此时如果窗口大小等于s1长度，说明窗口内字符就是s1的变位词。
一次窗口扩大就对应窗口的收缩判断，如果右窗口读到不是s1内的字符时，会将使这个字符计数变为-1，左窗口会直接收缩到右窗口位置，即加回-1计数，此时窗口内没有s1不需要的字符。

C++代码：

```C++
class Solution {
public:
    bool checkInclusion(string s1, string s2) {
        int m = s1.size();
        int n = s2.size();
        if (m > n) {
            return false;
        }
        // key: s1需要的字符， value：需要字符的个数
        unordered_map<char, int> mp;
        // 记下需要的字符
        for (char c: s1) {
            ++mp[c];
        }
        int left = 0;
        int right = 0;
        while (right < n) {
            // 右窗口扩大，减掉需要字符的计数。
            char c = s2[right];
            --mp[c];
            // 左窗口是否要收缩
            while (mp[c] < 0) {
                ++mp[s2[left]];// 加回需要的字符的计数
                ++left;
            }
            // 长度满足要求，直接返回，说明存在变位词
            if (right - left + 1 == m) {
                return true;
            }
            ++right;
        }
        return false;
    }
};
```

Java代码：

```Java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        // 如果s1长度大于s2长度，s2不可能存在s1的变位词
        if (m > n) {
            return false;
        }
        // 只有小写字符，用数组代替HashMap
        // 下标: s1需要的字符的映射， 值：需要字符的个数
        int[] need = new int[26];
        for (int i = 0; i < m; ++i) {
            ++need[s1.charAt(i) - 'a'];
        }
        int left = 0;
        int right = 0;
        while (right < n) {
            // 右窗口扩大，减掉需要字符的计数。
            char c1 = s2.charAt(right);
            --need[c1 - 'a'];
            // 左窗口是否要收缩
            while (need[c1 - 'a'] < 0) {
                char c2 = s2.charAt(left);
                ++need[c2 - 'a']; // 加回需要的字符的计数
                ++left;
            }
            // 长度满足要求，直接返回，说明存在变位词
            if (right - left + 1 == m) {
                return true;
            }
            ++right;
        }
        return false;
    }
}
```

### 015字符串中的所有变位词

方法：同014题

C++代码：

```C++
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> res;
        int m = s.size();
        int n = p.size();
        if (n > m) {
            return res;
        }
        // key: s1需要的字符， value：需要字符的个数
        unordered_map<char, int> mp;
        // 记下需要的字符
        for (char c: p) {
            ++mp[c];
        }
        int left = 0;
        int right = 0;
        while (right < m) {
            // 右窗口扩大，减掉需要字符的计数。
            char c = s[right];
            --mp[c];
            // 左窗口是否要收缩
            while (mp[c] < 0) {
                ++mp[s[left]];// 加回需要的字符的计数
                ++left;
            }
            // 长度满足要求，说明存在变位词, 记下起始索引，即窗口左侧
            if (right - left + 1 == n) {
                // 与014唯一不同，这里记下起始索引，不直接返回
                res.push_back(left);    
            }
            ++right;
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> res = new ArrayList<>();
        int m = p.length();
        int n = s.length();
        // 如果s1长度大于s2长度，s2不可能存在s1的变位词
        if (m > n) {
            return res;
        }
        // 只有小写字符，用数组代替HashMap
        // 下标: s1需要的字符的映射， 值：需要字符的个数
        int[] need = new int[26];
        for (int i = 0; i < m; ++i) {
            ++need[p.charAt(i) - 'a'];
        }
        int left = 0;
        int right = 0;
        while (right < n) {
            // 右窗口扩大，减掉需要字符的计数。
            char c1 = s.charAt(right);
            --need[c1 - 'a'];
            // 左窗口是否要收缩
            while (need[c1 - 'a'] < 0) {
                char c2 = s.charAt(left);
                ++need[c2 - 'a']; // 加回需要的字符的计数
                ++left;
            }
            // 长度满足要求，直接返回，说明存在变位词
            if (right - left + 1 == m) {
                // 与014唯一不同，这里记下起始索引，不直接返回
                res.add(left);
            }
            ++right;
        }
        return res;
    }
}
```

### 016 不含重复字符的最长子字符串

方法：滑动窗口，窗口右边扩大直到窗口内出现一个重复元素，记下此时的窗口大小-1，也就是不包含重复字符的连续子串长度。然后窗口左边收缩，直到排出窗口右侧的重复元素，此时窗口内无重复元素。以此循环

C++代码：

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int res = 0;
        // 记录窗口内的元素，用于判断是否出现重复
        unordered_set<char> set;
        int left = 0;
        int right = 0;
        int n = s.size();
        while (right < n) {
            // 窗口右边扩大直到窗口内出现一个重复元素
            while (right < n && set.count(s[right]) == 0) {
                set.insert(s[right]);
                ++right;
            }
            // 此时窗口大小-1为此时不包含重复字符的连续子串长度，
            // （一般窗口大小为right - left + 1)
            res = max(res,right - left);
            // 然后窗口左边收缩，直到排出窗口右侧的重复元素
            while (left < right && set.count(s[right]) == 1) {
                set.erase(s[left]);
                ++left;
            }
        }
        return res;
    }
};
```

Java代码：

```Java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int res = 0;
        // 记录窗口内的元素，用于判断是否出现重复
        Set<Character> set = new HashSet<>();
        int left = 0;
        int right = 0;
        int n = s.length();
        while (right < n) {
            // 窗口右边扩大直到窗口内出现一个重复元素
            while (right < n && !set.contains(s.charAt(right))) {
                set.add(s.charAt(right));
                ++right;
            }
            // 此时窗口大小-1为此时不包含重复字符的连续子串长度，
            // （一般窗口大小为right - left + 1)
            res = Math.max(res, right - left);
            // 然后窗口左边收缩，直到排出窗口右侧的重复元素
            while (left < right && right < n && set.contains(s.charAt(right))) {
                set.remove(s.charAt(left));
                ++left;
            }
        }
        return res;
    }
}
```

### 017含有所有字符的最短字符串

方法：滑动窗口，先用HashMap记下需要的字符和每个字符对应个数，然后使用另一个HashMap当作滑动窗口。如果窗口左侧的字符不是需要的字符，或者窗口左侧的那个字符的数量多于需要的字符，窗口左侧收缩。窗口左侧收缩后，遍历需要字符的HashMap，和窗口内的字符比较，如果满足包含条件，则记下子串的起始位置和长度，供结果返回。

C++代码：

```C++
class Solution {
public:
    string minWindow(string s, string t) {
        // 如果s串的长度更短，则不可能存在符合条件子串
        if (s.size() < t.size()) {
            return "";
        }
        // key：字符串中每个字符， value：字符个数
        unordered_map<char, int> need; // 需要的字符计数
        unordered_map<char, int> window;// 滑动窗口内的字符计数
        // 先记下需要的字符及个数
        for (char c: t) {
            ++need[c];
        }
        int idx = 0;    // 答案子串的起始位置
        int length = INT_MAX;   // 答案子串的长度
        int left = 0; 
        int right = 0;
        int n = s.size();
        while (right < n) {
            char c = s[right];
            ++window[c];
            // 如果窗口左侧的字符不是需要的字符，或者窗口左侧的那个字符的数量多于需要的字符，窗口左侧收缩
            while (left < right && (need.find(s[left]) == need.end() || window[s[left]] > need[s[left]])) {
                --window[s[left]];
                ++left;
            }
            // flag标记窗口内的字符是否满足，满足则记下起始坐标和长度
            bool flag = true;
            for (auto& [ch, cnt]: need) {
                // 如果需要的字符不再窗口中，或者窗口内字符个数少于需要的个数，则不符合要求
                if (window.find(ch) == window.end() || cnt > window[ch]) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                // 如果窗口更短，重新记下子串下标和长度
                if (length > right - left + 1) {
                    length = right - left + 1;
                    idx = left;
                }
            }
            ++right;
        }
        // 如果长度为INT_MAX,说明没有找到符合条件子串
        return length == INT_MAX? "": s.substr(idx, length);
    }
};
```

Java代码

```Java
class Solution {
    public String minWindow(String s, String t) {
        // 如果s串的长度更短，则不可能存在符合条件子串
        if (s.length() < t.length()) {
            return "";
        }
        // key：字符串中每个字符， value：字符个数
        Map<Character, Integer> need = new HashMap<>();// 需要的字符计数
        Map<Character, Integer> window = new HashMap<>();// 滑动窗口内的字符计数
        // 先记下需要的字符及个数
        for (int i = 0; i < t.length(); ++i) {
            need.put(t.charAt(i), need.getOrDefault(t.charAt(i), 0) + 1);
        }
        int begin = 0;    // 答案子串的起始位置
        int length = Integer.MAX_VALUE; // 答案子串的长度
        int left = 0;
        int right = 0;
        int n = s.length();
        while (right < n) {
            char c1 = s.charAt(right);
            window.put(c1, window.getOrDefault(c1, 0) + 1);
            // 如果窗口左侧的字符不是需要的字符，或者窗口左侧的那个字符的数量多于需要的字符，窗口左侧收缩
            while (left < right && (!need.containsKey(s.charAt(left)) || window.get(s.charAt(left)) > need.get(s.charAt(left)))) {
                char c2 = s.charAt(left);
                window.put(c2, window.get(c2) - 1);
                ++left;
            }
            // flag标记窗口内的字符是否满足，满足则记下起始坐标和长度
            boolean flag = true;
            for (Map.Entry<Character, Integer> entry: need.entrySet()) {
                char ch = entry.getKey();   // 需要的字符
                int cnt = entry.getValue(); // 需要字符的个数
                // 如果需要的字符不在窗口中，或者窗口内字符个数少于需要的个数，则不符合要求
                if (!window.containsKey(ch) || cnt > window.get(ch)) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                // 如果窗口更短，重新记下子串下标和长度
                if (length > right - left + 1) {
                    length = right - left + 1;
                    begin = left;
                }
            }
            ++right;
        }
        // 如果长度为INT_MAX,说明没有找到符合条件子串
        return length == Integer.MAX_VALUE? "": s.substring(begin, begin + length);
    }
}
```

### 018有效的回文

方法：使用双指针，从左边和右边同时向中间靠拢，如果是需要忽略的字符，先跳过

C++代码：

```C++
class Solution {
public:
    // C++ 中的tolower() 函数将给定字符转换为小写。它在cctype 头文件中定义。
    // C 库函数 void isalnum(int c) 检查所传的字符是否是字母和数字。
    bool isPalindrome(string s) {
        int left = 0;
        int right = s.size()-1;
        while (left < right) {
            // 跳过左边的忽略字符
            while (left < right && !isalnum(s[left])) {
                ++left;
            }
            // 跳过右边的忽略字符
            while (left < right && !isalnum(s[right])) {
                --right;
            }
            if (left < right) {
                // 全部转为小写，再比较是否相当，不相等说明不是回文串
                if (tolower(s[left]) != tolower(s[right])) {
                    return false;
                }
                ++left;
                --right;
            }
        }   
        return true; 
    }
};
```

Java代码

```Java
class Solution {
    public boolean isPalindrome(String s) {
        int left = 0;
        int right = s.length() - 1;
        while (left < right) {
            // 跳过左边的忽略字符
            while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
                ++left;
            }
            // 跳过右边的忽略字符
            while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
                --right;
            }
            if (left < right) {
                // 全部转为小写，再比较是否相当，不相等说明不是回文串
                if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
                    return false;
                }
                ++left;
                --right;
            }
        }
        return true;
    }
}
```

### 019 最多删除一个字符得到回文

方法：双指针，指向字符串头尾。两边同时向中间靠拢，如果出现了不相同的字符，考虑删除左边指针的一个字符或者删除右边指针的一个字符（实际应用时，跳过当作删除），分别判断剩下的字符是否是回文串。如果删除左边指针指向字符后的字符串是回文，或删除右边指针指向字符后的字符串是回文，这满足最多删除一个字符能得到回文串。

C++代码：

```C++
class Solution {
public:
    bool validPalindrome(string s) {
        int left = 0;
        int right = s.size()-1;
        // 出现不同字符使，判断删除左指针指向字符或删除右指针指向的字符之后，是否是回文串。
        bool resLeft = true;  
        bool resRight = true;
        while (left < right) {
            // 两边相同时，向中间靠拢。
            if (s[left] == s[right]) {
                ++left;
                --right;
            } else {
                // 两边出现了一个不同字符时，分别判断
                // 删除（跳过）左指针字符，进行向中间考虑判断
                int i = left + 1;
                int j = right;
                while (i < j) {
                    // 出现不相同字符，不是回文串，直接结束循环
                    if (s[i++] != s[j--]) {
                        resLeft = false;
                        break;
                    }
                }
                // 删除（跳过）右指针字符，进行向中间考虑判断
                i = left;
                j = right - 1;
                while (i < j) {
                    // 出现不相同字符，不是回文串，直接结束循环
                    if (s[i++] != s[j--]) {
                        resRight = false;
                        break;
                    }
                }
                // 删除一个还是回文，则满足题目条件
                return resRight || resLeft;
            }
        }
        // 不删除一个字符串也是回文
        return true;
    }
};
```

Java代码

```Java
class Solution {
    public boolean validPalindrome(String s) {
        int left = 0;
        int right = s.length() - 1;
        // 出现不同字符使，判断删除左指针指向字符或删除右指针指向的字符之后，是否是回文串。
        boolean resLeft = true;
        boolean resRight = true;
        while (left < right) {
            // 两边相同时，向中间靠拢。
            if (s.charAt(left) == s.charAt(right)) {
                ++left;
                --right;
            } else {
                // 两边出现了一个不同字符时，分别判断
                // 删除（跳过）左指针字符，进行向中间考虑判断
                int i = left + 1;
                int j = right;
                while (i < j) {
                    // 出现不相同字符，不是回文串，直接结束循环
                    if (s.charAt(i) != s.charAt(j)) {
                        resLeft = false;
                        break;
                    }
                    ++i;
                    --j;
                }
                // 删除（跳过）右指针字符，进行向中间考虑判断
                i = left;
                j = right - 1;
                while (i < j) {
                    // 出现不相同字符，不是回文串，直接结束循环
                    if (s.charAt(i) != s.charAt(j)) {
                        resRight = false;
                        break;
                    }
                    ++i;
                    --j;
                }
                // 删除一个还是回文，则满足题目条件
                return resRight || resLeft;
            }
        }
        // 不删除一个字符串也是回文
        return true;
    }
}
```

### 020 回文子字符串的个数

方法：双指针，从遍历字符串每一个位置，从每一个位置开始，同时向两边展开，展开过程有两种情况，如果是奇数，则以当前位置为轴向相比展开；如果是偶数，则以这个位置和这个位置右边为中心，同时向两边展开。

C++代码：

```C++
class Solution {
public:
    int countSubstrings(string s) {
        int res = 0;
        int n = s.size();
        // 奇数字符个数的子串有n个中心轴，偶数字符个数的子串有n-1个中心
        // 遍历次数为 2 * n - 1， 这样在遍历过程中，除2，奇数后偶数的操作变为一致
        for (int i = 0; i < 2*n-1; ++i) {   
            // left 和right要么相同（奇数）， 要么相邻（偶数）
            int left = i / 2;
            int right = i / 2 + i % 2;
            while (left >= 0 && right < n && s[left] == s[right]) {
                --left;
                ++right;
                ++res;
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int countSubstrings(String s) {
        int n = s.length();
        int res = 0;
        // 奇数字符个数的子串有n个中心轴，偶数字符个数的子串有n-1个中心
        // 遍历次数为 2 * n - 1， 这样在遍历过程中，除2，奇数后偶数的操作变为一致
        for (int i = 0; i < n * 2 - 1; ++i) {
            // left 和right要么相同（奇数）， 要么相邻（偶数）
            int left = i / 2;
            int right = i / 2 + i % 2;
            while (left >= 0 && right < n && s.charAt(left) == s.charAt(right)) {
                --left;
                ++right;
                ++res;
            }
        }
        return res;
    }
}
```

## 链表

链表常用方法为双指针，或者快慢指针。
技巧：

* 找中间节点的一个技巧为两个指针，一个一次两步、一个一次一步，块指针到末尾时，慢指针指向重点
* 可以使用头结点之前的一个节点接上头结点，使对头结点的操作和其他所有节点一致
* 注意使用头插法和尾插法，头插法可以起到使节点逆序重排的作用

### 021 删除链表的倒数第n个节点

方法：建立一个头结点使头结点和其他节点操作一致，使用两个指针，一个指针先移动N步，另一个指针再和快指针同时移动，前一个指针到末尾了，另一个指针就是指向倒数第N个节点

C++代码：

```C++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // 头前节点指向头结点
        ListNode* first = new ListNode(0, head);
        ListNode* fast = first;
        ListNode* lastN = first;
        // 一个指针先移动N步
        for (int i = 0; i < n; ++i) {
            fast = fast->next;
        }
        // 两指针同时移动
        // 这里是fast->next到达末尾，此时另一个指针指向倒数第一个节点的前一个节点，这样方便删除
        while (fast->next != nullptr) {
            lastN = lastN->next;
            fast = fast->next;
        }
        // 删除节点，即直接断开该结点的连接即可
        lastN->next = lastN->next->next;
        // 返回原头结点
        return first->next;
    }
};
```

Java代码

```Java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // 头前节点指向头结点
        ListNode preHead = new ListNode(0, head);
        ListNode fast = preHead;
        ListNode lastN = preHead;
        // 一个指针先移动N步
        for (int i = 0; i < n; ++i) {
            fast = fast.next;
        }
        // 两指针同时移动
        // 这里是fast->next到达末尾，此时另一个指针指向倒数第一个节点的前一个节点，这样方便删除
        while (fast.next != null) {
            lastN = lastN.next;
            fast = fast.next;
        }
        // 删除节点，即直接断开该结点的连接即可
        lastN.next = lastN.next.next;
        return preHead.next;
    }
}
```

### 022 链表环的入口节点

方法：快慢指针，快指针一次走两格，慢指针一次走一格。如果快指针到链表末端，说明链表无环。否则，因为每走一次，快指针和慢指针的间距+1，快指针终会追上慢指针，即到达同一个点。
设头结点到入口距离为$a$，环的大小为$b$， 入口到相遇的点的距离为$l$，此时相遇点的另一个方向到入口的距离为$b-l$。此时，慢指针走了$a+l$，快指针走了$a+l+nb$，快指针走的距离是慢指针两倍，有$2a+2l = a + l + nb$，于是有$a = (n-1)b + (b-l)$，即头结点到入口的距离为$n-1$圈加上相遇点到入口点的距离。此时一个指针指向头结点，和慢指针同步移动，当慢指针走了$(n-1)b + (b-l)$步时，另一个指针也走了这么多步，即走了$a$步，他们会在入口点相遇。

C++代码：

```C++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        // 头指针为空或只有一个结点，不可能有环
        if (head == NULL || head->next == NULL) {
            return NULL;
        }
        ListNode* slow = head->next;
        ListNode* fast = head->next->next;
        while (slow != fast) {
            // 快指针到了末尾，无环
            if (fast == NULL || fast->next==NULL) {
                return NULL;
            }
            // 快慢指针
            slow = slow->next;
            fast = fast->next->next;
        }
        // 另一个指针指向头结点，两指针同时移动，交点即为入口点
        ListNode* res = head;
        while (res != slow) {
            res = res->next;
            slow = slow->next;
        }
        return res;
    }
};
```

Java代码

```Java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        // 头指针为空或只有一个结点，不可能有环
        if (head == null || head.next == null) {
            return null;
        }
        ListNode slow = head.next;
        ListNode fast = head.next.next;
        while (slow != fast) {
            // 快指针到了末尾，无环
            if (fast == null || fast.next == null) {
                return null;
            }
            // 快慢指针
            slow = slow.next;
            fast = fast.next.next;
        }
        // 另一个指针指向头结点，两指针同时移动，交点即为入口点
        ListNode res = head;
        while (res != slow) {
            res = res.next;
            slow = slow.next;
        }
        return res;
    }
}
```

### 023 两个链表的第一个重合结点

方法：双指针，两个指针容两个头结点同时移动，当到达末尾时，从另一个头结点开始继续移动，最终两个指针会在交点相遇，如果没有交点，两个指针都会指向末尾的null

C++代码：

```C++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (headA == NULL || headB == NULL) {
            return NULL;
        }
        ListNode* nodeA = headA;
        ListNode* nodeB = headB;
        while (nodeA != nodeB) {
            if (nodeA == NULL) {
                // 从A开始移动的节点到达末尾，继续从B开始移动
                nodeA = headB;
            } else {
                nodeA = nodeA->next;
            }
            if (nodeB == NULL) {
                // 从B开始移动的节点到达末尾，继续从A开始移动
                nodeB = headA;
            } else {
                nodeB = nodeB->next;
            }  
        }
        // 最终两个指针会在交点相遇，如果没有交点，两个指针都会指向末尾的null
        return nodeA;
    }
};
```

Java代码

```Java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }
        ListNode pA = headA;
        ListNode pB = headB;
        while (pA != pB) {
            if (pA == null) {
                // 从A开始移动的节点到达末尾，继续从B开始移动
                pA = headB;
            } else {
                pA = pA.next;
            }
            if (pB == null) {
                // 从B开始移动的节点到达末尾，继续从A开始移动
                pB = headA;
            } else {
                pB = pB.next;
            }
        }
        // 最终两个指针会在交点相遇，如果没有交点，两个指针都会指向末尾的null
        return pA;
    }
}
```

### 024 反转链表

方法：建立一个头结点之前的结点指向头结点，使用头插法将链表反转

C++代码：

```C++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        // 新建一个结点，再此结点上使用头插法
        ListNode* preHead = new ListNode();
        // head指针一直向后移动
        while (head != nullptr) {
            ListNode* tmp = head->next;   // 记下当前的下一个结点
            head->next = preHead->next;     // 进行头插
            preHead->next = head;   
            head = tmp; // 向下移动一个节点
        }
        return preHead->next;
    }
};
```

Java代码

```Java
class Solution {
    public ListNode reverseList(ListNode head) {
        // 新建一个结点，再此结点上使用头插法
        ListNode preHead = new ListNode();
        ListNode cur = head;    // 指向当前要操作的节点
        while (cur != null) {
            ListNode tmp = cur.next;// 记下当前的下一个结点
            cur.next = preHead.next; // 进行头插
            preHead.next = cur;
            cur = tmp;// 向下移动一个节点
        }
        // 返回反转后的头结点
        return preHead.next;
    }
}
```

### 025 链表中的两数相加

方法：使用两个栈将两个链表结点（或值）放到栈中，然后新建一个链表，使用头插法，从栈中逐个弹出计算，同时维护一个进位位进行下次计算。如果一个栈为空，则弹出0进行计算，知道两个链表都为空。最后再判断是否还有进位，有的话记得加上。

C++代码：

```C++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* preHead = new ListNode();
        stack<ListNode*> st1;
        stack<ListNode*> st2;
        // 先把两个链表的所有结点放到两个栈中
        while(l1 != nullptr) {
            st1.push(l1);
            l1 = l1->next;
        }
        while (l2 != nullptr) {
            st2.push(l2);
            l2 = l2->next;
        }
        int c = 0;  // 进位位
        // 逐个弹出计算
        while (!st1.empty() || !st2.empty()) {
            int val1 = 0;   // 如果链表为空，弹出的为0
            if (!st1.empty()) {
                val1 = st1.top()->val;
                st1.pop();
            }
            int val2 = 0;   // 如果链表为空，弹出的为0
            if (!st2.empty()) {
                val2 = st2.top()->val;
                st2.pop();
            }
            int val = val1 + val2 + c;  // 对应两点相加
            // 如果有进位，则需要记下
            if (val >= 10) {
                c = 1;
                val = val % 10;
            } else {
                c = 0;
            }
            // 头插法插入新结点
            ListNode* node = new ListNode(val);
            node->next = preHead->next;
            preHead->next = node;
        }
        // 如果最后还有进位，需要加上进位位
        if (c != 0) {
            ListNode* node = new ListNode(1);
            node->next = preHead->next;
            preHead->next = node;  
        }
        // 返回实际头结点
        return preHead->next;
    }
};
```

Java代码

```Java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode preHead = new ListNode();
        Deque<ListNode> stack1 = new ArrayDeque<>();
        Deque<ListNode> stack2 = new ArrayDeque<>();
        // 先把两个链表的所有结点放到两个栈中
        while (l1 != null) {
            stack1.push(l1);
            l1 = l1.next;
        }
        while (l2 != null) {
            stack2.push(l2);
            l2 = l2.next;
        }
        int c = 0;  // 进位位
        // 逐个弹出计算
        while (!stack1.isEmpty() || ! stack2.isEmpty()) {
            int val1 = 0;   // 如果链表为空，弹出的为0
            if (!stack1.isEmpty()) {
                val1 = stack1.pop().val;
            }
            int val2 = 0;   // 如果链表为空，弹出的为0
            if (!stack2.isEmpty()) {
                val2 = stack2.pop().val;
            }
            int val = val1 + val2 + c;  // 对应两点相加
            // 如果有进位，则需要记下
            if (val >= 10) {
                c = 1;
                val = val % 10;
            } else {
                c = 0;
            }
            // 头插法插入新结点
            ListNode node = new ListNode(val);
            node.next = preHead.next;
            preHead.next = node;
        }
        // 如果最后还有进位，需要加上进位位
        if (c != 0) {
            ListNode node = new ListNode(1);
            node.next = preHead.next;
            preHead.next = node;
        }
        // 返回实际头结点
        return preHead.next;
    }
}
```

### 026 重排链表

方法：找到中间结点，再反转后半部分的链表，然后两部分链表交替连接。

C++代码：

```C++
class Solution {
public:
    void reorderList(ListNode* head) {
        ListNode* l1 = head;
        ListNode* midNode = findMid(head);  // 找到中间结点
        //反转后半部分，注意这里从中间结点的后一个结点开始
        ListNode* l2 = reverseList(midNode->next);  
        midNode->next = nullptr;    // 为了方便判断，断开前半部分的末尾连接
        // 交替连接重排链表
        while (l1 != nullptr && l2 != nullptr) {
            ListNode* tmp1 = l1->next;  // 记下下一个结点
            ListNode* tmp2 = l2->next;
            l1->next = l2;  // 连到另一部分结点上
            l1 = tmp1;  // 移动到下一个结点
            l2->next = l1;
            l2 = tmp2;
        }
    }

    // 使用快慢指针找到中间结点
    ListNode* findMid(ListNode* head) {
        ListNode* fast = head;
        ListNode* slow = head;
        while (fast->next != nullptr && fast->next->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;
        }
        return slow;
    }

    // 反转链表
    ListNode* reverseList(ListNode* head) {
        ListNode* ahead = new ListNode();
        while (head != nullptr) {
            ListNode* tmp = head->next;
            head->next = ahead->next;
            ahead->next = head;
            head = tmp;
        }
        return ahead->next;
    }
};
```

Java代码

```Java
class Solution {
    public void reorderList(ListNode head) {
        ListNode l1 = head;
        ListNode midNode = findMid(head); // 找到中间结点
        //反转后半部分，注意这里从中间结点的后一个结点开始
        ListNode l2 = reverseList(midNode.next);
        midNode.next = null;    // 为了方便判断，断开前半部分的末尾连接
        // 交替连接重排链表
        while (l1 != null && l2 != null) {
            ListNode tmp1 = l1.next;    // 记下下一个结点
            ListNode tmp2 = l2.next;
            l1.next = l2;   // 连到另一部分结点上
            l1 = tmp1;   // 移动到下一个结点
            l2.next = l1;
            l2 = tmp2;
        }
    }

    // 使用快慢指针找到中间结点
    ListNode findMid(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;
        while (fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        return slow;
    }

    // 反转链表
    public ListNode reverseList(ListNode head) {
        // 新建一个结点，再此结点上使用头插法
        ListNode preHead = new ListNode();
        ListNode cur = head;    // 指向当前要操作的节点
        while (cur != null) {
            ListNode tmp = cur.next;// 记下当前的下一个结点
            cur.next = preHead.next; // 进行头插
            preHead.next = cur;
            cur = tmp;// 向下移动一个节点
        }
        // 返回反转后的头结点
        return preHead.next;
    }
}
```

### 027 回文链表

方法：找到中间结点，反转后半部分结点，然后从中间和头结点同时开始向后逐个判断。

C++代码：

```C++
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        ListNode* slow = head;
        ListNode* fast = head;
        // 快慢指针找到中间结点
        while (fast != nullptr && fast->next != nullptr) {
            fast = fast->next->next;
            slow = slow->next;
        }
        // 如果快指针为空，说明结点个数是偶数个
        // 如果快指针的下一个结点为空，说明结点个数是奇数个
        if (fast != nullptr) {
            slow = slow->next;      // 奇数个节点，从下一个开始比较
        }
        // 反转后半部分
        ListNode* l2 = reverseList(slow);
        ListNode* l1 = head;
        // 同时向后移动进行判断
        while (l2 != nullptr) {
            if (l1->val != l2->val) {
                return false;
            }
            l1 = l1->next;
            l2 = l2->next;
        }
        return true;
    }

    // 反转链表
    ListNode* reverseList(ListNode* head) {
        ListNode* ahead = new ListNode();
        while (head != nullptr) {
            ListNode* tmp = head->next;
            head->next = ahead->next;
            ahead->next = head;
            head = tmp;
        }
        return ahead->next;
    }
};
```

Java代码

```Java
class Solution {
    public boolean isPalindrome(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        // 快慢指针找到中间结点
        while (fast != null && fast.next != null) {
            fast = fast.next.next;
            slow = slow.next;
        }
        // 如果快指针为空，说明结点个数是偶数个
        // 如果快指针的下一个结点为空，说明结点个数是奇数个
        if (fast != null) {
            slow = slow.next;   // 奇数个节点，从下一个开始比较
        }
        // 反转后半部分
        ListNode l2 = reverseList(slow);
        ListNode l1 = head;
        // 同时向后移动进行判断
        while (l2 != null) {
            if (l1.val != l2.val) {
                return false;
            }
            l1 = l1.next;
            l2 = l2.next;
        }
        return true;
    }

    // 反转链表
    public ListNode reverseList(ListNode head) {
        // 新建一个结点，再此结点上使用头插法
        ListNode preHead = new ListNode();
        ListNode cur = head;    // 指向当前要操作的节点
        while (cur != null) {
            ListNode tmp = cur.next;// 记下当前的下一个结点
            cur.next = preHead.next; // 进行头插
            preHead.next = cur;
            cur = tmp;// 向下移动一个节点
        }
        // 返回反转后的头结点
        return preHead.next;
    }
}
```

### 028 展平多级双向链表

方法：

C++代码：

```C++
class Solution {
public:
    Node* flatten(Node* head) {
        dfs(head);
        return head;
    }

    // 深度优先遍历，每次返回每一级链表下的最后一个结点
    Node* dfs(Node* node) {
        Node* last = node;  // 用来指示本次循环，也就是本级链表的最后一个结点
        while (node != nullptr) {
            if (node->child == nullptr) {
                // 没有分级结点，往后遍历
                last = node;
                node = node->next;
            } else {
                // 存在下一级链表
                Node* tmp = node->next; // 记录上一级结点的后一个结点
                Node* childLast = dfs(node->child); // 进入递归，获得下一级结点的最后一个结点
                // 连接下一级结点的头部
                node->next = node->child;   // 用上一级结点连上从他分出的下一级结点
                node->child->prev = node;   // 双向连接
                node->child = nullptr;  // 将分级的指针置为空
                // 连接下一级的尾部
                if (childLast != nullptr) {
                    // 如果下一级的结点不为空，则连接上一级分叉位置
                    // 也就是相当于接入上一级链表的中间
                    childLast ->next = tmp;
                }
                if (tmp != nullptr) {
                    // 双向链表，需要往前连接
                    tmp->prev = childLast;
                }
                last = node;    // 记录最后一个结点，这个在前面，才能保证node为null时正常返回
                node = childLast;   // 直接移动到接入的最后一个结点
            }
        }
        return last;
    }
};
```

Java代码

```Java
class Solution {
    public Node flatten(Node head) {
        dfs(head);
        return head;
    }

    // 深度优先遍历，每次返回每一级链表下的最后一个结点
    Node dfs(Node node) {
        Node last = node;   // 用来指示本次循环，也就是本级链表的最后一个结点
        while (node != null) {
            if (node.child == null) {
                // 没有分级结点，往后遍历
                last = node;
                node = node.next;
            } else {
                // 存在下一级链表
                Node tmp = node.next;   // 记录上一级结点的后一个结点
                Node childLast = dfs(node.child);   // 进入递归，获得下一级结点的最后一个结点
                // 连接下一级结点的头部
                node.next = node.child;// 用上一级结点连上从他分出的下一级结点
                node.child.prev = node;// 双向连接
                node.child = null;  // 将分级的指针置为空
                // 连接下一级的尾部
                if (childLast != null) {
                    childLast.next = tmp;
                }
                if (tmp != null) {
                    tmp.prev = childLast;
                }
                last = node;    // 记录最后一个结点，这个在前面，才能保证node为null时正常返回
                node = childLast;   // 直接移动到接入的最后一个结点
            }
        }
        return last;
    }
}
```

### 029 排序的循环链表

方法：首先判断循环链表结点的个数，只有0个或1个结点直接插入返回。然后使用两个指针同时移动，找到插入位置，插入位置分三种情况：1：插入值在两指针之间。 2：前指针的值大于后指针的值，且插入的值大于前指针的值，此时插入数据为链表最大值，插入到两指针之间。3：前指针的值大于后指针的值，且插入的值小于后指针的值，此时插入数据为链表最小，插入到两指针之间。2和3情况时前后指针指向原有最大值和最小值。

C++代码：

```C++
class Solution {
public:
    Node* insert(Node* head, int insertVal) {
        Node* ins = new Node(insertVal);    // 新建插入结点
        // 如果链表为空，插入结点自己连接上返回
        if (head == NULL) {
            ins->next = ins;
            return ins;
        }
        // 如果只有1个结点，插入后返回。
        if (head->next == head) {
            head->next = ins;
            ins->next = head;
            return head;
        }
        // 当前结点从头开始遍历
        Node* cur = head;
        // 结束条件是插入值在当前指针和下一指针之间。
        while (!(cur->next->val > insertVal && cur->val <= insertVal)) {
            cur = cur->next;
            Node* tmp = cur;
            // 如果出现重复值，往后到最后一个重复值。后一个条件防止全是重复值导致的无限循环
            while (cur->val == cur->next->val && cur->next != tmp) {
                cur = cur->next;
            }
            // 到了链表中最大值和最小值位置，则可以直接在这中间插入
            if (cur->next->val <= cur->val && (insertVal >= cur->val || insertVal <= cur->next->val)) {
                break;
            }
        }
        // 插入结点
        ins->next = cur->next;
        cur->next = ins;
        return head;
    }
};
```

Java代码

```Java
class Solution {
    public Node insert(Node head, int insertVal) {
        Node ins = new Node(insertVal); // 新建插入结点
        // 如果链表为空，插入结点自己连接上返回
        if (head == null) {
            ins.next = ins;
            return ins;
        }
        // 如果只有1个结点，插入后返回。
        if (head.next == head) {
            head.next = ins;
            ins.next = head;
            return head;
        }
        // 当前结点从头开始遍历
        Node cur = head;
        // 结束条件是插入值在当前指针和下一指针之间。
        while (!(cur.next.val >= insertVal && cur.val <= insertVal)) {
            cur = cur.next;
            Node tmp = cur;
            // 如果出现重复值，往后到最后一个重复值。后一个条件防止全是重复值导致的无限循环
            while (cur.val == cur.next.val && cur.next != tmp) {
                cur = cur.next;
            }
            // 到了链表中最大值和最小值位置，则可以直接在这中间插入
            if (cur.next.val <= cur.val && (insertVal >= cur.val || insertVal <= cur.next.val)) {
                break;
            }
        }
        // 插入结点
        ins.next = cur.next;
        cur.next = ins;
        return head;
    }
}
```

## 哈希表

### 030 插入、删除和随机访问都是O(1)的容器

方法：使用一个可随机访问的可变容器（C++ vector， Java List）作为基本容器，然后使用一个HashMap记录插入的数字以及下标。在插入时，插入到容器的最后一个位置，并记下下标；删除时，用删除数的位置和最后一个位置交换，然后弹出最后一个位置即可。访问是就访问在容器大小内的一个随机下标。

C++代码：

```C++
class RandomizedSet {
private:
    unordered_map<int, int> map;// key插入的数，value下标
    vector<int> nums;// 存放的基本容器
public:
    // 看了答案能轻松默写出来
    /** Initialize your data structure here. */
    RandomizedSet() {

    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    bool insert(int val) {
        if (map.count(val)) {
            // 已经存在
            return false;
        }
        // 插入到最后一个位置
        int n = nums.size();
        nums.push_back(val);
        map[val] = n;   // 记录插入的数以及位置
        return true;
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    bool remove(int val) {
        if (!map.count(val)) {
            // 不存在
            return false;
        }
        int idx = map[val]; // 获取删除数的下标
        int last = nums.back(); // 获取最后一个数的数值
        nums[idx] = last;   // 用最后一个数覆盖，相当于删除了这个数
        map[last] = idx;    // 重新记下原来最后一个数的新下标
        map.erase(val); // 删除HashMap的记录
        nums.pop_back();    // 弹出最后一个数
        return true;

    }
    
    /** Get a random element from the set. */
    int getRandom() {
        // 在容器大小访问随机访问
        return nums[rand()%nums.size()];
    }
};
```

Java代码

```Java
class RandomizedSet {
    public List<Integer> nums;// 存放的基本容器
    public Map<Integer, Integer> map;// key插入的数，value下标
    Random random;

    /** Initialize your data structure here. */
    public RandomizedSet() {
        nums = new ArrayList<>();// 存放的基本容器
        map = new HashMap();// key插入的数，value下标
        random = new Random();
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    public boolean insert(int val) {
        if (map.containsKey(val)) {
            // 已经存在
            return false;
        }
        // 插入到最后一个位置
        int n = nums.size();
        nums.add(val);
        map.put(val, n);    // 记录插入的数以及位置
        return true;
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    public boolean remove(int val) {
        if (!map.containsKey(val)) {
            // 不存在
            return false;
        }
        int idx = map.get(val);// 获取删除数的下标
        int last = nums.get(nums.size()-1);// 获取最后一个数的数值
        nums.set(idx, last);// 用最后一个数覆盖，相当于删除了这个数
        map.put(last, idx); // 重新记下原来最后一个数的新下标
        map.remove(val);// 删除HashMap的记录
        nums.remove(nums.size()-1);// 弹出最后一个数
        return true;
    }
    
    /** Get a random element from the set. */
    public int getRandom() {
        return nums.get(random.nextInt(nums.size()));
    }
}
```

### 031 最近最少使用缓存

方法：使用HashMap + 双向链表，HashMap记录链表中的每个结点，键为链表结点的键值，值为链表结点的指针（引用），这样能直接根据要访问的键找到要找的结点，使用双向链表也是便于结点的增加移动和删除。在每次获取键对应值时，将该结点移动到链表头部。当每次放进新的键值时，如果链表已经存在，将其移动到链表头部，否则，在两边头部插入数据，增加链表大小的计数，并记录到HashMap中，此时如果链表大小超过缓存大小，从链表尾部移除元素

C++代码：

```C++
// 构建双向链表
struct DLinkedNode {
    int key;
    int val;
    DLinkedNode* prev;
    DLinkedNode* next;
    // 有参构造
    DLinkedNode(int key, int val):key(key), val(val){

    }
    // 无参构造
    DLinkedNode() {

    }
};

class LRUCache {
public:
    DLinkedNode* head;
    DLinkedNode* tail;
    int capacity;   // 缓存容量
    int size;   // 链表大小（不包含头尾结点）
    unordered_map<int, DLinkedNode*> mp; // key: node point

    // 初始化双向链表
    LRUCache(int capacity): size(0), capacity(capacity) {
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head->next = tail;
        tail->prev = head;
    }
    
    int get(int key) {
        if (mp.find(key) != mp.end()) {
            // 如果存在，获取后移动到头部
            moveToHead(mp[key]);
            return mp[key]->val;
        }
        return -1;
    }
    
    void put(int key, int value) {
        if (mp.find(key) == mp.end()) { // 缓存不存在key
            DLinkedNode* node = new DLinkedNode(key, value);
            mp[key] = node; // HashMap记录下key和对应的结点
            addToHead(node);    // 移动到头部
            ++size;
            if (size > capacity) {
                // 如果链表大小大于缓存容量，删除尾部元素
                DLinkedNode* removed = removeTail();
                mp.erase(removed->key); // 从HashMap删除记录
                delete removed;
                --size;
            }
        } else {
            // 如果存在，获取后直接移动到头部
            mp[key]->val = value;
            moveToHead(mp[key]);
        }
    }

    // 在链表头部添加结点
    void addToHead(DLinkedNode* node) {
        node->next = head->next;
        head->next->prev = node;
        head->next = node;
        node->prev = head;
    }

    // 从链表中移除结点
    void removeNode(DLinkedNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    // 将结点移动到头部，相当于删除该结点，添加一个相同结点到头部
    void moveToHead(DLinkedNode* node) {
        removeNode(node);
        addToHead(node);
    }

    // 删除尾部结点，需要返回尾部结点，方便在HashMap中删除，也可以在此函数删除
    DLinkedNode* removeTail() {
        DLinkedNode* node = tail->prev;
        removeNode(node);
        return node;
    }
};
```

Java代码

```Java
// 构建双向链表
class DLinkedNode {
    public int key;
    public int val;
    public DLinkedNode prev;
    public DLinkedNode next;
    public DLinkedNode(){

    }
    public DLinkedNode(int key, int val) {
        this.key = key;
        this.val = val;
    }
}

class LRUCache {
    private DLinkedNode head;
    private DLinkedNode tail;
    private int capacity;
    private int size;
    private Map<Integer, DLinkedNode> map;

    public LRUCache(int capacity) {
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head.next = tail;
        tail.prev = head;
        this.size = 0;
        this.capacity = capacity;
        map = new HashMap<>();
    }
    
    public int get(int key) {
        if (map.containsKey(key)) {
            // 如果存在，获取后移动到头部
            DLinkedNode node = map.get(key);
            moveToHead(node);
            return node.val;
        }
        return -1;
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) {
            // 如果存在，获取后直接移动到头部
            DLinkedNode node = map.get(key);
            node.val = value;
            moveToHead(node);
        } else {
            // 缓存不存在key
            DLinkedNode node = new DLinkedNode(key, value);
            map.put(key, node); // HashMap记录下key和对应的结点
            addToHead(node); // 移动到头部
            ++size;
            if (size > capacity) {
                // 如果链表大小大于缓存容量，删除尾部元素
                DLinkedNode removed = removeTail();
                map.remove(removed.key); // 从HashMap删除记录
                --size;
            }
        }
    }

    // 在链表头部添加结点
    private void addToHead(DLinkedNode node) {
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
        node.prev = head;
    }

    // 从链表中移除结点
    private void removeNode(DLinkedNode node) {
        node.next.prev = node.prev;
        node.prev.next = node.next;
    }

    // 将结点移动到头部，相当于删除该结点，添加一个相同结点到头部
    private void moveToHead(DLinkedNode node) {
        removeNode(node);
        addToHead(node);
    }

    // 删除尾部结点，需要返回尾部结点，方便在HashMap中删除，也可以在此函数删除
    DLinkedNode removeTail() {
        DLinkedNode node = tail.prev;
        removeNode(node);
        return node;
    }
}
```

### 032 有效的变位词

方法：先记下所有字母的出现次数，本题只有26字母所以用固定数组就可以，对一个字符串，每出现一个字母，就再数组对应位置加1， 对于另一个字符串，每出现一个字母，就在数组相对应位置减1，最后判断数组内是否所有位置都为0.

C++代码：

```C++
class Solution {
public:
    bool isAnagram(string s, string t) {
        // 两字符串不一样长，一定不是变位词，直接返回
        if (s == t || s.size() != t.size()) {
            return false;
        }
        // 一个数组记下两个字符串出现次数的差值
        vector<int> mp(26, 0);
        for (int i = 0; i < s.size(); ++i) {
            ++mp[s[i] - 'a'];
            --mp[t[i] - 'a'];
        }
        // 遍历数组判断是够都为0
        for (int i = 0; i < 26; ++i) {
            if (mp[i] != 0) {
                return false;
            }
        }
        return true;
    }
};
```

Java代码

```Java
class Solution {
    public boolean isAnagram(String s, String t) {
        // 两字符串不一样长，一定不是变位词，直接返回
        if (s.equals(t) || s.length() != t.length()) {
            return false;
        }
        // 一个数组记下两个字符串出现次数的差值
        int[] cnt = new int[26];
        for (int i = 0; i < s.length(); ++i) {
            ++cnt[s.charAt(i) - 'a'];
            --cnt[t.charAt(i) - 'a'];
        }
        // 遍历数组判断是够都为0
        for (int i = 0; i < 26; ++i) {
            if (cnt[i] != 0) {
                return false;
            }
        }
        return true;
    }
}
```

### 033 变位词组

方法：变位词所包含的字符和字符个数相同，所以可以用变位词的字母及其个数作为键，例如a2b1这样的键，使用一个HashMap的键存放每一种变位词，其值为对应的变位词列表。

C++代码：

```C++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // key：同一个变位词的标识组合字符串。value:变位词列表。
        unordered_map<string, vector<string>> mp;
        // 对于每一个字符串，取出他们的固定标识作为键
        for (string& str: strs) {
            // 26个小写字母计数
            vector<int> cs(26, 0);
            for (char c: str) {
                ++cs[c - 'a'];
            }
            // 用计数来拼接一个唯一的键
            string key;
            for (int i = 0; i < 26; ++i) {
                if (cs[i] != 0) {
                    key += (i + 'a');
                    key += cs[i];   // 使用能够标识就可以，不一定要是数字
                }
            }
            // 将字符串放进对应键的列表
            mp[key].push_back(str);

        }
        // 从HashMap中取出变位词组放到答案中
        vector<vector<string>> res;
        for (auto [_, v]: mp) {
            res.push_back(v);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        // key：同一个变位词的标识组合字符串。value:变位词列表。
        Map<String, List<String>> map = new HashMap<>();
        // 对于每一个字符串，取出他们的固定标识作为键
        for (String str: strs) {
            // 26个小写字母计数
            int[] cs = new int[26];
            for (int i = 0; i < str.length(); ++i) {
                ++cs[str.charAt(i) - 'a'];
            }
            // 用计数来拼接一个唯一的键
            StringBuffer keyBuf = new StringBuffer();
            for (int i = 0; i < 26; ++i) {
                if (cs[i] != 0) {
                    keyBuf.append((char)(i + 'a'));
                    keyBuf.append(cs[i]);
                }
            }
            // 将字符串放进对应键的列表
            String key = keyBuf.toString();
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        // 返回HashMap内所有的值
        return new ArrayList<List<String>>(map.values());
    }
}
```

### 034 外星语言是否排序

方法：先用一个字典记下外星语言的字母顺序，然后使用这个字典映射，就相当于还原了原来的字母顺序。然后，对所有单词两两比较。

C++代码：

```C++
class Solution {
public:
    bool isAlienSorted(vector<string>& words, string order) {
        int n = words.size();
        // 记下外星语言的字母顺序，之后使用这个字典，相当于还原了正常字母顺序
        vector<int> v(26);
        for (int i = 0; i < 26; ++i) {
            v[order[i] - 'a'] = i;
        }
        // 对所有单词两两判断
        for (int i = 1; i < n; ++i) {
            bool flag = true; // 用于判断是否一个词是另一个的前缀
            for (int j = 0; j < words[i].size() && j < words[i-1].size(); ++j) {
                if (v[words[i][j]-'a'] < v[words[i-1][j]-'a']) {
                    // 如果字符顺序不符合字典序直接返回
                    return false;
                } else if (v[words[i][j]-'a'] > v[words[i-1][j]-'a']) {
                    // 符合字典序，跳出循环
                    flag = false;   // 此时没有前缀关系
                    break;
                }
            }
            // 如果一个词是另一个词前缀，则判断前一个词长度是否比后一个长
            if (flag) {
                if (words[i].size() < words[i-1].size()) {
                    return false;
                }
            }
        }
        return true;
    }
};
```

Java代码

```Java
class Solution {
    public boolean isAlienSorted(String[] words, String order) {
        int n = words.length;
        // 记下外星语言的字母顺序，之后使用这个字典，相当于还原了正常字母顺序
        int[] dict = new int[26];
        for (int i = 0; i < order.length(); ++i) {
            dict[order.charAt(i) - 'a'] = i;
        }
        // 对所有单词两两判断
        for (int i = 1; i < n; ++i) {
            boolean flag = true; // 用于判断是否一个词是另一个的前缀
            for (int j = 0; j < words[i].length() && j < words[i-1].length(); ++j) {
                if (dict[words[i].charAt(j) - 'a'] < dict[words[i-1].charAt(j) - 'a']) {
                    // 如果字符顺序不符合字典序直接返回
                    return false;
                } else if (dict[words[i].charAt(j) - 'a'] > dict[words[i-1].charAt(j) - 'a']) {
                    // 符合字典序，跳出循环
                    flag = false;   // 此时没有前缀关系
                    break;
                }
            }
            // 如果一个词是另一个词前缀，则判断前一个词长度是否比后一个长
            if (flag) {
                if (words[i].length() < words[i-1].length()) {
                    return false;
                }
            }
        }
        return true;
    }
}
```

### 035 最小时间差

方法：将时间戳排序后，最小时间差必然出现在两个相邻的时间，或者两个首尾时间中。排序后遍历一遍即可

C++代码：

```C++
class Solution {
public:
    int findMinDifference(vector<string>& timePoints) {
        int res = INT_MAX;
        int n = timePoints.size();
        if (n > 1440) { // 如果时间点大于60 X 24 = 1440，必然出现重复时间点。
            return 0;
        }
        // 排序时间，一样格式的数据，字符串排序顺序恰好和整型一样
        sort(timePoints.begin(), timePoints.end());
        // 两两计算时间差
        int time = getMinutes(timePoints[0]);
        for (int i = 1; i < n; ++i) {
            int cur = getMinutes(timePoints[i]);
            res = min(res, cur - time);
            time = cur;
        }
        // 最小相邻时间差和首尾时间差进行比较
        return min(res, getMinutes(timePoints[0]) - getMinutes(timePoints[n-1]) + 1440);
    }

    // 将时间变换成从00:00开始的分钟数。
    int getMinutes(string& t) {
        return (int(t[0] - '0') * 10 + int(t[1] - '0')) * 60 + int(t[3] - '0') * 10 + int(t[4] - '0');
    }
};
```

Java代码

```Java
class Solution {
    public int findMinDifference(List<String> timePoints) {
        int res = Integer.MAX_VALUE;
        int n = timePoints.size();
        // 如果时间点大于60 X 24 = 1440，必然出现重复时间点。
        if (n > 1440) {
            return 0;
        }
        // 排序时间，一样格式的数据，字符串排序顺序恰好和整型一样
        Collections.sort(timePoints);
        // 两两计算时间差
        for (int i = 1; i < n; ++i) {
            int time1 = getMinutes(timePoints.get(i));
            int time2 = getMinutes(timePoints.get(i-1));
            res = Math.min(res, time1 - time2);
        }
        // 最小相邻时间差和首尾时间差进行比较
        return Math.min(res, getMinutes(timePoints.get(0)) - getMinutes(timePoints.get(n-1)) + 1440);
    }

    // 将时间变换成从00:00开始的分钟数。
    public int getMinutes(String t) {
        return ((t.charAt(0) - '0') * 10 + (t.charAt(1) - '0')) * 60
                 + (t.charAt(3) - '0') * 10 + (t.charAt(4) - '0');
    }
}
```

## 栈

常用方法：
[单调栈](https://leetcode.cn/problems/largest-rectangle-in-histogram/solution/bao-li-jie-fa-zhan-by-liweiwei1419/)：单调栈是一种暴力解法的优化方法，用于找到对应数值呈单调形态。

```C++
while () {
    while (!stack.empty() && nums[stk.top()] > nums[i]) {
        ...
        stack.pop();
        ...
    }
    ...
    stack.push(i);
    ...
}
```

### 036 后缀表达式

方法：遍历表达式，如果是数字，则放到栈中，如果是运算符，则取出两个数进行运算，再将结果放到栈中。最后的结果就是栈中的唯一一个（位于栈顶）元素

C++代码：

```C++
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<int> stk;
        for (string s: tokens) {
            if (s == "+" || s == "-" || s == "*" || s=="/") {
                // 如果是运算符，取出两个数进行运算
                int num1 = stk.top();
                stk.pop();
                int num2 = stk.top();
                stk.pop();
                // 根据运算符进行计算
                if (s == "+") {
                    stk.push(num2 + num1);
                } else if (s == "-") {
                    stk.push(num2 - num1);
                } else if (s == "*") {
                    stk.push(num2 * num1);
                } else if (s == "/") {
                    stk.push(num2 / num1);
                }
            } else {
                // 如果是数字，直接放到栈中
                stk.push(stoi(s));
            }
        }
        return stk.top();
    }
};
```

Java代码

```Java
class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> stack = new LinkedList<>();
        for (String token: tokens) {
            if ("+".equals(token) || "-".equals(token) || "*".equals(token) || "/".equals(token)) {
                 // 如果是运算符，取出两个数进行运算
                int num2 = stack.pop();// 注意顺序
                int num1 = stack.pop();
                // 根据运算符进行计算
                switch(token) {
                    case "+":
                        stack.push(num1 + num2);
                        break;
                    case "-":
                        stack.push(num1 - num2);
                        break;
                    case "*":
                        stack.push(num1 * num2);
                        break;
                    case "/":
                        stack.push(num1 / num2);
                        break;
                }
            } else {
                // 如果是数字，直接放到栈中
                stack.push(Integer.parseInt(token));
            }
        }
        return stack.pop();
    }
}
```

### 037 小行星碰撞

方法：

C++代码：

```C++

```

Java代码

```Java

```

### 038 每日温度

方法：

C++代码：

```C++

```

Java代码

```Java

```

### 039 直方图最大矩形面积

方法：

C++代码：

```C++

```

Java代码

```Java

```

### 040 矩阵中最大的矩形

方法：

C++代码：

```C++

```

Java代码

```Java

```
