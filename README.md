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
for (int i = 0; i < n; ++i) 或
while (i < n) {
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

方法：使用栈模拟碰撞，对于每个遍历到的小行星，使用一个变量记录它会不会爆炸。如果小行星存在且向负方向移动，且栈顶元素向正方向移动，则两行星会碰撞，留下大的行星，知道本行星被比它大的击碎或击碎所有致栈顶为空。其他情况，不会碰撞，直接放入栈中。

C++代码：

```C++
class Solution {
public:
    vector<int> asteroidCollision(vector<int>& asteroids) {
        vector<int> st;
        for (int a: asteroids) {
            bool alive = true;
            // 如果小行星存在且向负方向移动，且栈顶元素向正方向移动
            while (alive && a < 0 && !st.empty() && st.back() > 0) {
                alive = st.back() < -a; // 判断哪个行星更大，即当前能否存活
                // 如果存活，将栈顶行星击碎，等号是同归于尽
                if (st.back() <= -a) {
                    st.pop_back();
                }
            }
            // 如果还存活，放到栈中。
            if (alive) {
                st.push_back(a);
            }
        }
        return st;
    }
};
```

Java代码

```Java
class Solution {
    public int[] asteroidCollision(int[] asteroids) {
        Deque<Integer> stack = new LinkedList<>();
        for (int a: asteroids) {
            boolean alive = true;
            // 如果小行星存在且向负方向移动，且栈顶元素向正方向移动
            while(alive && a < 0 && !stack.isEmpty() && stack.peek() > 0) {
                alive = stack.peek() < -a;// 判断哪个行星更大，即当前能否存活
                // 如果存活，将栈顶行星击碎，等号是同归于尽
                if (stack.peek() <= -a) {
                    stack.pop();
                }
            }
            // 如果还活着，放到栈中
            if (alive) {
                stack.push(a);
            }
        }
        // 将栈中的元素，逆着放回答案数组
        int size = stack.size();
        int[] res = new int[size];
        for (int i = size - 1; i >= 0; --i) {
            res[i] = stack.pop();
        }
        return res;
    }
}
```

### 038 每日温度

方法：使用单调栈，栈中温度递减，栈内存放温度数组的下标（日期），当温度更高时，逐个弹出并记下每个弹出的日期距更高温度的距离

C++代码：

```C++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        vector<int> res(n);
        stack<int> st;
        for (int i = 0; i < n; ++i) {
            // 当温度不减，出现更高温度
            while (!st.empty() && temperatures[i] > temperatures[st.top()]) {
                int idx = st.top();
                res[idx] = i - idx;// 记下栈中弹出的日期距离此时更高温度的距离
                st.pop();
            }
            st.push(i);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] res = new int[n];
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < n; ++i) {
            // 当温度不减，出现更高温度
            while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
                int idx = stack.pop();
                res[idx] = i - idx;// 记下栈中弹出的日期距离此时更高温度的距离
            }
            stack.push(i);
        }
        return res;
    }
}
```

### 039 直方图最大矩形面积

方法：对于每一列，找到以本列的高度，能够向两边扩展的最远距离，此时可以计算本列能够构成的矩形面积，对每一列计算能够构成的矩形面积，就能得到最大的面积。对于如何找到向两边扩展的距离，类似上一题，求比它低的第一个数距本列的距离，对左右两边分别使用单调栈，存到两个数组中。

C++代码：

```C++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int res = 0;
        int n = heights.size();
        stack<int> st;
        // 分别记下左边和右边可延伸到的位置,默认左边为数组头前一个位置，右边为数组尾后一个位置
        vector<int> left(n);
        vector<int> right(n, n);
        for (int i = 0; i < n; ++i) {
            // 单调栈，当栈内元素不递增时，记下左边的列距离当前列距离，
            // 当前列也就是左边列能向右扩展的最大距离
            while (!st.empty() && heights[i] < heights[st.top()]) {
                right[st.top()] = i;
                st.pop();
            }
            // 因为单调栈的递增性质，当前下标左边的延伸位置，是栈顶的记录的下标
            // 这里也可以类似上面，再进行一次从右往左遍历记下左边的延伸位置
            left[i] = st.empty()? -1: st.top();
            st.push(i);
        }
        // 对每一列，计算能够向两边构造的矩形面积
        for (int i = 0; i < n; ++i) {
            res = max(res, (right[i] - left[i] - 1) * heights[i]);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        // 分别记下左边和右边可延伸到的位置,默认左边为数组头前一个位置，右边为数组尾后一个位置
        int[] left = new int[n];
        int[] right = new int[n];
        Arrays.fill(right, n);
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < n; ++i) {
            // 单调栈，当栈内元素不递增时，记下左边的列距离当前列距离，
            // 当前列也就是左边列能向右扩展的最大距离
            while (!stack.isEmpty() && heights[i] < heights[stack.peek()]) {
                int idx = stack.pop();
                right[idx] = i;
            }
            // 因为单调栈的递增性质，当前下标左边的延伸位置，是栈顶的记录的下标
            // 这里也可以类似上面，再进行一次从右往左遍历记下左边的延伸位置
            left[i] = stack.isEmpty()? -1: stack.peek();
            stack.push(i);
        }
        int res = 0;
        // 对每一列，计算能够向两边构造的矩形面积
        for (int i = 0; i < n; ++i) {
            res = Math.max(res, (right[i] - left[i] - 1) * heights[i]);
        }
        return res;
    }
}
```

### 040 矩阵中最大的矩形

方法：对于每一行，计算每一行作为直方图x轴的最大矩形面积。

C++代码：

```C++
class Solution {
public:
    int maximalRectangle(vector<string>& matrix) {
        int m = matrix.size();
        if (m == 0) {
            return 0;
        }
        int n = matrix[0].size();
        int res = 0;
        vector<int> heights(n, 0);
        // 从第0行开始，计算有1构成的直方图的高度
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i][j] == '1') {
                    heights[j] += matrix[i][j] - '0';
                } else {
                    heights[j] = 0;
                }
            }
            // 以当前行作为x坐标轴，计算直方图最大矩形面积
            res = max(res, largestRectangleArea(heights));
        }   
        return res;
    }

    // 上一题计算直方图最大矩形面积完全相同的代码
    int largestRectangleArea(vector<int>& heights) {
        int res = 0;
        int n = heights.size();
        stack<int> st;
        vector<int> left(n);
        vector<int> right(n, n);
        for (int i = 0; i < n; ++i) {
            while (!st.empty() && heights[i] < heights[st.top()]) {
                right[st.top()] = i;
                st.pop();
            }
            left[i] = st.empty()? -1: st.top();
            st.push(i);
        }
        for (int i = 0; i < n; ++i) {
            res = max(res, (right[i] - left[i] - 1) * heights[i]);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int maximalRectangle(String[] matrix) {
        int m = matrix.length;
        if (m == 0) {
            return 0;
        }
        int n = matrix[0].length();
        int res = 0;
        int[] heights = new int[n];
        // 从第0行开始，计算有1构成的直方图的高度
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i].charAt(j) == '1') {
                    heights[j] += matrix[i].charAt(j) - '0';
                } else {
                    heights[j] = 0;
                }
            }
            // 以当前行作为x坐标轴，计算直方图最大矩形面积
            res = Math.max(res, largestRectangleArea(heights));
        }   
        return res;
    }

    // 上一题计算直方图最大矩形面积完全相同的代码
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        int[] left = new int[n];
        int[] right = new int[n];
        Arrays.fill(right, n);
        Deque<Integer> stack = new ArrayDeque<>();
        for (int i = 0; i < n; ++i) {
            while (!stack.isEmpty() && heights[i] < heights[stack.peek()]) {
                int idx = stack.pop();
                right[idx] = i;
            }
            left[i] = stack.isEmpty()? -1: stack.peek();
            stack.push(i);
        }
        int res = 0;
        for (int i = 0; i < n; ++i) {
            res = Math.max(res, (right[i] - left[i] - 1) * heights[i]);
        }
        return res;
    }
}
```

## 队列

常用于二叉树的层次遍历

```C++
queue.push(root)
while (!queue.empty()) {
    int n =queue.size();
    for (int i = 0; i < n; ++i) {
        TreeNode* node = queue.front();
        queue.pop();
        ...
        if (node->left != nullptr) {
            queue.push(node->left);
        }
        ...
        if (node->right != nullptr) {
            queue.push(node->right);
        }
        ...
    }
}
```

### 041 滑动窗口的平均值

方法：使用一个队列计算这个窗口，用一个变量记录窗口内元素的和，每次滑动，总和减去移除的值，加上移入的值，求平均值用总和除队列大小即可

C++代码：

```C++
class MovingAverage {
public:
    /** Initialize your data structure here. */
    queue<int> q;   // 队列
    int size;   // 滑动窗口的大小
    double sum; // 队列元素和
    MovingAverage(int size): size(size), sum(0) {
    }
    
    double next(int val) {
        if (q.size() < size) {
            // 队列大小小于窗口，只加入元素
            q.push(val);
            sum += val;
            return sum / q.size();
        } else {
            // 队列大小达到窗口大小，加入一个移除一个
            sum -= q.front();
            q.pop();
            sum += val;
            q.push(val);
        }
        return sum / size;
    }
};
```

Java代码

```Java
class MovingAverage {
    public int size;
    public double sum;
    Queue<Integer> queue;
    /** Initialize your data structure here. */
    public MovingAverage(int size) {
        this.queue = new ArrayDeque<Integer>();
        this.size = size;
        this.sum = 0;
    }
    
    public double next(int val) {
        // 队列大小达到窗口大小，移除一个
        if (queue.size() == size) {
            sum -= queue.poll();
        }
        queue.offer(val);
        sum += val;
        return sum / queue.size();
    }
}
```

### 042 最近请求次数

方法：使用一个队列记下每次请求，当队头的元素时间小于当前时间减3000，则出队

C++代码：

```C++
class RecentCounter {
public:
    queue<int> q;
    RecentCounter() {
    }
    
    int ping(int t) {
        q.push(t);
        // 当队头的元素时间小于当前时间减3000，则出队
        while (q.front() < t - 3000) {
            q.pop();
        }
        return q.size();
    }
};
```

Java代码

```Java
class RecentCounter {
    public Queue<Integer> queue;
    public RecentCounter() {
        queue = new ArrayDeque<>();
    }
    
    public int ping(int t) {
        queue.offer(t);
        // 当队头的元素时间小于当前时间减3000，则出队
        while (queue.peek() < t - 3000) {
            queue.poll();
        }
        return queue.size();
    }
}
```

### 043 往完全二叉树添加结点

方法：使用一个队列按照层次遍历的方式，存放完全二叉树的叶子节点或只有左子树的节点，在往完全二叉树添加结点时，从队列头取出元素作为父节点，接上添加的结点，并将此结点也入队

C++代码：

```C++
class CBTInserter {
public:
    TreeNode* root;
    queue<TreeNode*> candidate; // 候选插入结点的父节点队列
    CBTInserter(TreeNode* root): root(root) {
        queue<TreeNode*> q;
        q.push(root);
        // 按层次遍历，将完全二叉树最后几个可以插入的节点加入队列
        while (!q.empty()) {
            TreeNode* node = q.front();
            q.pop();
            if (node->left != nullptr) {
                q.push(node->left);
            }
            if (node->right != nullptr) {
                q.push(node->right);
            }
            // 叶节点或右节点为空的节点
            if (node->left == nullptr || node->right == nullptr) {
                candidate.push(node);
            }
        }
    }
    
    int insert(int v) {
        TreeNode* node = new TreeNode(v);
        // 队头是插入结点的父节点
        TreeNode* father = candidate.front();
        int res = father->val;
        if (father->left == nullptr) {
            // 左子树为空，是叶节点，直接连接
            father->left = node;
        } else {
            // 右节点为空，接上后此结点左右子树不空，弹出队列
            father->right = node;
            candidate.pop();
        }
        // 队列加入当前节点
        candidate.push(node);
        return res;
    }
    
    TreeNode* get_root() {
        return root;
    }
};
```

Java代码

```Java
class CBTInserter {
    public TreeNode root;
    Queue<TreeNode> candidate;// 候选插入结点的父节点队列
    public CBTInserter(TreeNode root) {
        this.root = root;
        candidate = new ArrayDeque<>();
        Queue<TreeNode> queue = new ArrayDeque<>();
        queue.offer(root);
        // 按层次遍历，将完全二叉树最后几个可以插入的节点加入队列
        while (!queue.isEmpty()) {
            TreeNode node = queue.poll();
            if (node.left != null) {
                queue.offer(node.left);
            }
            if (node.right != null) {
                queue.offer(node.right);
            }
            // 叶节点或右节点为空的节点
            if (node.left == null || node.right == null) {
                candidate.offer(node);
            }
        }
    }
    
    public int insert(int v) {
        TreeNode node = new TreeNode(v);
        // 队头是插入结点的父节点
        TreeNode father = candidate.peek();
        int res = father.val;
        if (father.left == null) {
            // 左子树为空，是叶节点，直接连接
            father.left = node;
        } else {
            // 右节点为空，接上后此结点左右子树不空，弹出队列
            father.right = node;
            candidate.poll();
        }
        // 队列加入当前节点
        candidate.offer(node);
        return res;
    }
    
    public TreeNode get_root() {
        return root;
    }
}
```

### 044 二叉树每层的最大值

方法：按层次遍历，在每层去最大值加到结果数组中

C++代码：

```C++
class Solution {
public:
    vector<int> largestValues(TreeNode* root) {
        queue<TreeNode*> q;
        q.push(root);
        vector<int> res;
        if (root == nullptr) {
            return res;
        }
        while (!q.empty()) {
            // 层次遍历，需要先记下每一层的大小
            int n = q.size();
            int maxVal = INT_MIN;
            for (int i = 0; i < n; ++i) {
                TreeNode* node = q.front();
                q.pop();
                // 记录每一层的最大值
                maxVal = max(maxVal, node->val);
                if (node->left != nullptr) {
                    q.push(node->left);
                }
                if (node->right != nullptr) {
                    q.push(node->right);
                }
            }
            // 遍历完一层，将最大值加到结果数组中
            res.push_back(maxVal);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public List<Integer> largestValues(TreeNode root) {
        Queue<TreeNode> queue = new ArrayDeque<>();
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        queue.offer(root);
        while (!queue.isEmpty()) {
            // 层次遍历，需要先记下每一层的大小
            int n = queue.size();
            int maxVal = Integer.MIN_VALUE;
            for (int i = 0; i < n; ++i) {
                TreeNode node = queue.poll();
                // 记录每一层的最大值
                maxVal = Math.max(maxVal, node.val);
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
            // 遍历完一层，将最大值加到结果数组中
            res.add(maxVal);
        }
        return res;
    }
}
```

### 045 二叉树最底层最左边的值

方法：层次遍历，每次都取每一层第一个名，到最后一层就是最后一层的第一个

C++代码：

```C++
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        int res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int n = q.size();
            for (int i = 0; i < n; ++i) {
                TreeNode* node = q.front();
                q.pop();
                // 每次都取每一层第一个名，到最后一层就是最后一层的第一个
                if (i == 0) {
                    res = node->val;
                }
                if (node->left != nullptr) {
                    q.push(node->left);
                }
                if (node->right != nullptr) {
                    q.push(node->right);
                }
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int findBottomLeftValue(TreeNode root) {
        Queue<TreeNode> queue = new ArrayDeque<>();
        int res = 0;
        queue.offer(root);
        while (!queue.isEmpty()) {
            // 层次遍历，需要先记下每一层的大小
            int n = queue.size();
            for (int i = 0; i < n; ++i) {
                TreeNode node = queue.poll();
                if (i == 0) {
                    res = node.val;
                }
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
        }
        return res;
    }
}
```

### 046 二叉树的右侧视图

方法：类似上一题，按从右往左的层次遍历，每次去每层的第一个节点

C++代码：

```C++
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> res;
        if (root == nullptr) {
            return res;
        }
        queue<TreeNode*> q;
        // 层次遍历
        q.push(root);
        while (!q.empty()) {
            int n = q.size();
            for (int i = 0; i < n; ++i) {
                TreeNode* node = q.front();
                q.pop();
                // 将每层第一个节点放入答案
                if (i == 0) {
                    res.push_back(node->val);
                }
                if (node->right != nullptr) {
                    q.push(node->right);
                }
                if (node->left != nullptr) {
                    q.push(node->left);
                }
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        Queue<TreeNode> queue = new ArrayDeque<>();
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        queue.offer(root);
        while (!queue.isEmpty()) {
            // 层次遍历，需要先记下每一层的大小
            int n = queue.size();
            for (int i = 0; i < n; ++i) {
                TreeNode node = queue.poll();
                if (i == n - 1) {
                    // 每一层的最右边节点
                    res.add(node.val);
                }
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
        }
        return res;
    }
}
```

## 树

四种遍历，中左右、左中右、左右中、层次遍历
无返回值和返回值为结点

### 047 二叉树剪枝

方法：后序遍历，左右中，逐步把为0的叶节点删除，每个结点的两个叶节点删除后，自己就变为页节点

C++代码：

```C++
class Solution {
public:
    TreeNode* pruneTree(TreeNode* root) {
        if (root == nullptr) {
            return nullptr;
        }
        root->left = pruneTree(root->left);
        root->right = pruneTree(root->right);
        // 遇到为零的叶节点，直接返回nullptr，相当于断开类这个节点
        if (root->left == nullptr && root->right == nullptr && root->val == 0) {
            return nullptr;
        }
        return root;
    }
};
```

Java代码

```Java
class Solution {
    public TreeNode pruneTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        root.left = pruneTree(root.left);
        root.right = pruneTree(root.right);
        // 遇到为零的叶节点，直接返回nullptr，相当于断开类这个节点
        if (root.left == null && root.right == null && root.val == 0) {
            return null;
        }
        return root;
    }
}
```

### 048 序列化与返序列化二叉树

方法：采用先序遍历（集中都可以），将二叉树存为字符，叶节点后跟#代表结束，用逗号分隔每个结点的数值，然后将序列变为字符串数组，同样的遍历方式遍历这个数组进行反序列化

C++代码：

```C++
class Codec {
public:
    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        string res;
        rserialize(root, res);
        return res;
    }

    // 先序遍历，序列化为字符串
    void rserialize(TreeNode* root, string& str) {
        if (root == NULL) {
            str += "#,";    // 用#表示结尾
        } else {
            str += to_string(root->val) + ",";  // 加上本节点的值，用逗号分隔
            rserialize(root->left, str);
            rserialize(root->right, str);
        }
    }

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        list<string> dataArray;
        string str; // 临时存放每一个节点的数组的字符串形式
        // 将字符串装换为数组
        for (char& c: data) {
            if (c == ',') {
                dataArray.push_back(str);
                str.clear();
            } else {
                str.push_back(c);
            }
        }
        // 最后一个结点的数组也要记得放进数组
        if (!str.empty()) {
            dataArray.push_back(str);
            str.clear();
        }
        return rdeserialize(dataArray);
    }

    // 使用字符串数组反序列化二叉树，递归时去除数组中已经反序列化完成的结点
    TreeNode* rdeserialize(list<string>& dataArray) {
        if (dataArray.front() == "#") {
            // 到达结尾，去除#，返回
            dataArray.erase(dataArray.begin());
            return NULL;
        }
        // 新建节点，先序遍历建立二叉树
        TreeNode* root = new TreeNode(stoi(dataArray.front()));
        dataArray.erase(dataArray.begin());
        root->left = rdeserialize(dataArray);
        root->right = rdeserialize(dataArray);
        return root;
    }
};
```

Java代码

```Java
public class Codec {
    // Encodes a tree to a single string.
    public String serialize(TreeNode root) {
        return rserialize(root, "");
    }

    // 先序遍历，序列化为字符串
    // 这里需要返回str，因为String不是单纯的引用传递
    public String rserialize(TreeNode root, String str) {
        if (root == null) {
            str += "#,"; // 用#表示结尾，注意要带逗号
        } else {
            str += String.valueOf(root.val) + ",";
            str = rserialize(root.left, str);
            str = rserialize(root.right, str);
        }
        return str;
    }

    // Decodes your encoded data to tree.
    public TreeNode deserialize(String data) {
        // 注意这里需要new，不然asList返回的是不可变对象
        List<String> dataArray = new ArrayList<>(Arrays.asList(data.split(",")));
        return rdserialize(dataArray);
    }

    public TreeNode rdserialize(List<String> dataArray) {
        if ("#".equals(dataArray.get(0))) {
            dataArray.remove(0);
            return null;
        }
        TreeNode root = new TreeNode(Integer.parseInt(dataArray.get(0)));
        dataArray.remove(0);
        root.left = rdserialize(dataArray);
        root.right = rdserialize(dataArray);
        return root;
    }
}
```

### 049 从根节点到叶节点的路径数字只和

方法：深度优先遍历，到叶节点时把数值加入总和。

C++代码：

```C++
class Solution {
public:
    int sum = 0;
    int sumNumbers(TreeNode* root) {
        dfs(root, 0);
        return sum;
    }

    void dfs(TreeNode* node, int cnt) {
        if (node == nullptr) {
            return ;
        }
        // 记录一个路径的值
        cnt = cnt*10 + node->val;
        dfs(node->left, cnt);
        dfs(node->right, cnt);
        // 到达叶节点，加入到总和
        if (node->left == nullptr && node->right == nullptr) {
            sum += cnt;
        }
    }
};
```

Java代码

```Java
class Solution {
    public int sumNumbers(TreeNode root) {
        return dfs(root, 0);
    }

    int dfs(TreeNode root, int preSum) {
        if (root == null) {
            return 0;
        }
        int sum = preSum * 10 + root.val;
        // 到达叶节点，返回一个路径的和
        if (root.left == null && root.right == null) {
            return sum;
        }
        // 计算左子树的路径和加上右子树的路径和
        return dfs(root.left, sum) + dfs(root.right, sum);
    }
}
```

### 050 向下的路径节点之和

方法：对于每一条根到叶的路径，使用类似 010 前缀和为k的子数组 的方法，计算出和为targetSum的路径数。使用一个HashMap，记录前缀和为某个数的个数，在遍历过程中，如果当前值-前缀和等于目标值，则可以直接加上该路径的数目

C++代码：

```C++
class Solution {
public:
    // 记录从根节点的前缀和
    // 注意本题需要用Long表示前缀和，否则数据会溢出
    unordered_map<long long, int> prefix;
    int pathSum(TreeNode* root, int targetSum) {
        // 初始条件，路径和为0的有1个
        prefix[0] = 1;
        return dfs(root, 0, targetSum);
    }

    int dfs(TreeNode* root, long long curr, int targetSum) {
        if (root == nullptr) {
            return 0;
        }
        int ret = 0;
        // 当前前缀和
        curr += root->val;
        // 如果当前值-前缀和等于目标值，则可以直接加上该路径的数目
        if (prefix.count(curr - targetSum)) {
            ret = prefix[curr - targetSum];
        }
        // 对当前前缀和计数
        prefix[curr]++;
        ret += dfs(root->left, curr, targetSum);
        ret += dfs(root->right, curr, targetSum);
        // 返回上一层时，需要减去此前缀和和计数，避免影响别的分支
        --prefix[curr];
        return ret;
    }
};
```

Java代码

```Java
class Solution {
    // 注意本题需要用Long表示前缀和，否则数据会溢出
    public Map<Long, Integer> map = new HashMap<>();
    public int pathSum(TreeNode root, int targetSum) {
        map.put(0L, 1);
        return dfs(root, targetSum, 0);
    }

    public int dfs(TreeNode root, int targetSum, long cur) {
        if (root == null) {
            return 0;
        }
        int ret = 0;
        // 当前前缀和
        cur += root.val;
        // 如果当前值-前缀和等于目标值，则可以直接加上该路径的数目
        ret += map.getOrDefault(cur - targetSum, 0);
        map.put(cur, map.getOrDefault(cur, 0) + 1);
        ret += dfs(root.left, targetSum, cur);
        ret += dfs(root.right, targetSum, cur);
        // 返回上一层时，需要减去此前缀和和计数，避免影响别的分支
        map.put(cur, map.get(cur) - 1);
        return ret;
    }
}
```

### 051 节点之和最大的路径

方法：在以每一个节点为根节点的子树中，寻找以该节点为起点的一条路径，使得该路径上的节点值之和最大。利用递归，计算左右子树的最大贡献值，每个结点的最大贡献值为当前节点的值，加上左边的贡献值或右边的贡献值。也就是说，从叶节点返回递归和过程，只计算某一条不分叉的路径给上一个结点

C++代码：

```C++
class Solution {
public:
    int res = INT_MIN;
    int maxPathSum(TreeNode* root) {
        maxGain(root);
        return res;
    }

    int maxGain(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }
        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        int left = max(0, maxGain(root->left));
        int right = max(0, maxGain(root->right));
        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        int sum = root->val + left + right;
        res = max(res, sum);    // 更新答案
        return root->val + max(left, right);    // 返回节点的最大贡献值
    }
};
```

Java代码

```Java
class Solution {
    public int res = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        maxGain(root);
        return res;
    }

    int maxGain(TreeNode root) {
        if (root == null) {
            return 0;
        }
        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        int left = Math.max(maxGain(root.left), 0);
        int right = Math.max(maxGain(root.right), 0);
        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        int sum = root.val + left + right;
        res = Math.max(res, sum);   // 更新答案
        return root.val + Math.max(left, right);    // 返回节点的最大贡献值
    }
}
```

### 052 展平二叉搜索树

方法：采用中序遍历（左中右）二叉搜索树可以获得升序序列，维护一个指针pre存放当前结点cur之前遍历的节点，然后之前的节点的左指针置为空，右指针连接当前节点，即可重排二叉搜索树。为了方便操作，事先新建一个结点作为最开始的节点。

C++代码：

```C++
class Solution {
public:
    TreeNode* pre = new TreeNode(-1);
    TreeNode* increasingBST(TreeNode* root) {
        TreeNode* tmp = pre;// 存放最开始的根之前的节点
        dfs(root);
        return tmp->right;
    }

    void dfs(TreeNode* curr) {
        if (curr == nullptr) {
            return;
        }
        dfs(curr->left);
        pre->right = curr;// 前一个指针右边连上当前
        curr->left = nullptr;// 将当前左指针置为空
        pre = curr;
        dfs(curr->right);
    }
};
```

Java代码

```Java
class Solution {
    TreeNode pre = new TreeNode();
    public TreeNode increasingBST(TreeNode root) {
        TreeNode preRoot = pre;// 存放最开始的根之前的节点
        dfs(root);// 前指针后移一个位置
        return preRoot.right;
    }

    // 中序遍历
    public void dfs(TreeNode cur) {
        if (cur == null) {
            return ;
        }
        dfs(cur.left);
        pre.right = cur;// 前一个指针右边连上当前
        cur.left = null;    // 将当前左指针置为空
        pre = cur;  // 前指针后移一个位置
        dfs(cur.right);
    }
}
```

### 053 二叉搜索树中的中序后继

方法：如果节点p右侧不为空，则它的中序后继是右子树最左侧节点；如果节点p右侧为空，它的中序后继就是比它大的最小节点，此结点可以用二叉搜索的方式搜索当前节点，且只有当前节点值大于p才更新结果，

C++代码：

```C++
class Solution {
public:
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {   
        // 如果p节点的右子树存在，它的中序后继是右子树最左侧节点
        if (p->right != NULL) {
            p = p->right;
            while (p->left != NULL) {
                p = p->left;
            }
            return p;
        }
        TreeNode* res = NULL;
        // 如果p节点的右子树不存在，以二叉搜索的方式
        while (root != NULL) {
            // 有当前节点值大于p才更新结果
            if (root->val > p->val) {
                res = root;
                root = root->left;
            } else {
                // 遍历到等于p时，会到p的右边空指针，结束循环
                // 或者遍历到p上一个节点，值都小于p，到叶节点从而结束循环
                root = root->right;
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public TreeNode inorderSuccessor(TreeNode root, TreeNode p) {
        TreeNode res = null;
        // 如果p节点的右子树存在，它的中序后继是右子树最左侧节点
        if (p.right != null) {
            res = p.right;
            while (res.left != null) {
                res = res.left;
            }
            return res;
        }
        // 如果p节点的右子树不存在，以二叉搜索的方式
        TreeNode cur = root;
        while (cur != null) {
            // 有当前节点值大于p才更新结果
            if (cur.val > p.val) {
                res = cur;
                cur = cur.left;
            } else {
                // 遍历到等于p时，会到p的右边空指针，结束循环
                // 或者遍历到p上一个节点，值都小于p，到右指针为空的结点从而结束循环
                cur = cur.right;
            }
        }
        return res;
    }
}
```

### 054 所有大于等于节点的值之和

方法：使用一个变量记录遍历到的节点的值之和，采用右中左的遍历方式，给每个结点用这个总和重新赋值，根据二叉搜索树特性，这样的值就是题目的要求

C++代码：

```C++
class Solution {
public:
    int sum = 0;
    TreeNode* convertBST(TreeNode* root) {
        dfs(root);
        return root;
    }

    // 右中左遍历
    void dfs(TreeNode* node) {
        if (node == nullptr) {
            return;
        }
        dfs(node->right);
        // 每次遍历到就加上
        sum += node->val;
        node->val = sum;
        dfs(node->left);
    }
};
```

Java代码

```Java
class Solution {
    int sum = 0;
    // 右中左遍历,只有最后一层返回值有效，递归过程中间的返回值无意义
    public TreeNode convertBST(TreeNode root) {
        if (root == null) {
            return null;
        }
        convertBST(root.right);
        sum += root.val;
        // 每次遍历到就加上
        root.val = sum;
        convertBST(root.left);
        return root;
    }
}
```

### 055 二叉搜索树迭代器

方法：对二叉树进行中序遍历，使用一个栈存放遍历的路径，每次获取下一个元素时，先向左子树遍历，存入路径，直到最左侧结点，然后从栈中弹出最左侧节点，获取其值后向当前节点右侧遍历一个位置。

C++代码：

```C++
class BSTIterator {
public:
    stack<TreeNode*> st;// 存放路径的栈
    TreeNode* p;    // 当前节点
    // 初始化时指向根节点
    BSTIterator(TreeNode* root): p(root) {
    }
    
    int next() {
        // 遍历到最左节点，也就是剩下的最小节点
        while (p != nullptr) {
            st.push(p);
            p = p->left;
        }
        // 取出最左节点
        p = st.top();
        st.pop(); 
        int ret = p->val;   // 获取剩下节点最小值
        p = p->right;   // 先当前结点右侧移动
        return ret;
    }
    
    bool hasNext() {
        // 指针不为空或者栈不为空，则还有下一个元素
        return p != nullptr || !st.empty(); // 最后一次成立的hasNext时，p不为空，栈为空，
    }
};
```

Java代码

```Java
class BSTIterator {
    private Deque<TreeNode> stack; // 存放路径的栈
    private TreeNode cur; // 当前节点
    // 初始化时指向根节点
    public BSTIterator(TreeNode root) {
        // 遍历到最左节点，也就是剩下的最小节点
        stack = new ArrayDeque<>();
        cur = root;
    }
    
    public int next() {
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
        // 取出最左节点
        cur = stack.pop();
        int ret = cur.val; // 获取剩下节点最小值
        cur = cur.right; // 先当前结点右侧移动
        return ret;
    }
    
    public boolean hasNext() {
        // 指针不为空或者栈不为空，则还有下一个元素
        return cur != null || !stack.isEmpty(); // 最后一次成立的hasNext时，p不为空，栈为空，
    }
}
```

### 056 二叉搜索树中两个节点只和

方法：使用中序遍历，遍历过程中用HashMap记下每个结点的值，在遍历过程中查找是否存在当前节点和HashMap中某一个值之和为k

C++代码：

```C++
class Solution {
public:
    unordered_set<int> set;
    bool findTarget(TreeNode* root, int k) {
        if (root == nullptr) {
            return false;
        }
        // Hash表中是否存在和当前值之和为k的值
        if (set.count(k - root->val)) {
            return true;
        }
        // hash表中记下当前结点的值
        set.insert(root->val);
        // 中序遍历
        return findTarget(root->left, k) || findTarget(root->right, k);
    }
};
```

Java代码

```Java
class Solution {
    private Set<Integer> set = new HashSet<>();
    public boolean findTarget(TreeNode root, int k) {
        if (root == null) {
            return false;
        }
        // Hash表中是否存在和当前值之和为k的值
        if (set.contains(k - root.val)) {
            return true;
        }
        // hash表中记下当前结点的值
        set.add(root.val);
        // 中序遍历
        return findTarget(root.left, k) || findTarget(root.right, k);
    }
}
```

### 057 值和下标之差都在给定的范围内

方法：使用hash表维护一个大小为k的滑动窗口，每次遍历一个元素时，窗口内包含当前元素之前之多k个元素，使用二分查找对hash表进行查找，看窗口内是否存在值在[cur-t, cur+t]区间的元素

C++代码：

```C++
class Solution {
public:
    bool containsNearbyAlmostDuplicate(vector<int>& nums, int k, int t) {
        // HashSet作为滑动窗口
        // 当前值加t的大小会超过int上限，需要用long来存方便比较
        set<long> st;   
        int n = nums.size();
        for (int i = 0; i < n; ++i) {
            auto it = st.lower_bound((long)nums[i] - t);    // 对HashSet进行二分查找
            // 窗口内存在符合条件元素，返回true
            if (it != st.end() && *it <= (long)nums[i] + t) {
                return true;
            }
            // 窗口右移
            st.insert(nums[i]);
            // 窗口大小超过k，移除最前面的元素
            if (i >= k) {
                st.erase(nums[i-k]);
            }
        }
        return false;
    }
};
```

Java代码

```Java
class Solution {
    public boolean containsNearbyAlmostDuplicate(int[] nums, int k, int t) {
        // HashSet作为滑动窗口
        // 当前值加t的大小会超过int上限，需要用long来存方便比较
        TreeSet<Long> set = new TreeSet<>();    // 有序set才能二分查找
        int n = nums.length;
        for (int i = 0; i < n; ++i) {
            Long ceiling = set.ceiling((long)nums[i] - (long)t);// 对HashSet进行二分查找
            // 窗口内存在符合条件元素，返回true, 因为是绝对值，需要两种判断
            if (ceiling != null  && ceiling <= (long)nums[i] + (long)t) {
                return true;
            }
            // 窗口右移
            set.add((long)nums[i]);
            // 窗口大小超过k，移除最前面的元素
            if (i >= k) {
                set.remove((long)nums[i-k]);
            }
        }
        return false;
    }
}
```

### 058 日程表

方法：使用有序的set，按起始时间排序记下每次的日程时间段，然后使用二分查找，找开始时间比本次结束时间晚一点的一个区间，看是否重复。一开始为空，或和这个区间的前一个区间不重复，说明不会重复

C++代码：

```C++
class MyCalendar {
public:
    set<pair<int, int>> st;
    MyCalendar() {

    }
    
    bool book(int start, int end) {
        auto it = st.lower_bound({end, 0}); // 找开始时间比本次结束时间晚一点的一个区间
        // 一开始还未预订begin==end，或最前面的开始时间比当前结束时间晚 
        // 或 前一个区间结束时间小于当前要订的开始时间，不会重复，没找到时，--st.end()，是最晚的一个区间
        if (it == st.begin() || (--it)->second <= start) {  
            st.insert(make_pair(start, end));
            return true;
        }
        return false;
    }
};
```

Java代码

```Java
class MyCalendar {
    TreeSet<int[]> booked;
    public MyCalendar() {
        // 自定义排序规则，即按开始时间排序
        booked = new TreeSet<>((a, b)-> a[0] - b[0]); 
    }
    
    public boolean book(int start, int end) {
        // 一开始没任何预订时，可直接预订
        if (booked.isEmpty()) {
            booked.add(new int[]{start, end});
            return true;
        }
        int[] arr = booked.ceiling(new int[]{end, 0});  // 大于当前结束时间的第一个日程
        // 如果不存在 大于当前结束时间的第一个日程， 则用最后一个作为当前的前一个，
        // 否则，找到小于arr的前一个
        int[] pre = arr == null ? booked.last(): booked.lower(arr);
        // 最前面的开始时间比当前结束时间晚 或 前一个区间结束时间小于当前要订的开始时间，不会重复
        if (arr == booked.first() || pre[1] <= start) {
            booked.add(new int[]{start, end});
            return true;
        }
        return false;
    }
}
```

## 堆

堆有内置的优先队列数据结构包，主要要掌握自定义比较器写法
[C++优先队列写法](https://blog.csdn.net/qq_21539375/article/details/122128445)
[java优先队列写法](https://docs.oracle.com/en/java/javase/18/docs/api/java.base/java/util/PriorityQueue.html)

### 059 数据流的第K大数值

方法：使用优先队列，维护一个大小为k的优先队列，使用小顶堆，这样堆顶的数字就是第K大的数值

C++代码：

```C++
class KthLargest {
public:
    priority_queue<int, vector<int>, greater<int>> pq; // 小顶堆
    int k;
    // 初始化堆大小，把已有数据流添加进优先队列
    KthLargest(int k, vector<int>& nums): k(k) {
        for (int& num: nums) {
            add(num);
        }
    }
    
    int add(int val) {
        pq.push(val);
        // 当优先队列大小大于k，弹出一个数
        if (pq.size() > k) {
            pq.pop();
        }
        // 堆顶就是第k大数值
        return pq.top();
    }
};
```

Java代码

```Java
class KthLargest {
    private PriorityQueue<Integer> pq; // 默认小顶堆
    private int k;
    // 初始化堆大小，把已有数据流添加进优先队列
    public KthLargest(int k, int[] nums) {
        this.k = k;
        pq = new PriorityQueue<>();
        for (int num: nums) {
            add(num);
        }
    }
    
    public int add(int val) {
        pq.add(val);
        // 当优先队列大小大于k，弹出一个数
        if (pq.size() > k) {
            pq.poll();
        }
        // 堆顶就是第k大数值
        return pq.peek();
    }
}
```

### 060 出现频率最高的k个数字

方法：先用HashMap记下所有数字出现的次数，然后使用按频率降序的优先队列，将HashMap中的记录放到优先队列中，最后弹出k个频率最高的k个数字

C++代码：

```C++
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> mp;
        // 字典记下所有数字出现频率
        for (auto& num: nums) {
            ++mp[num];
        }
        // 降序排列比较函数
        auto cmp = [](const pair<int, int>& a, const pair<int, int>& b) {
            return a.second < b.second;
        };
        // 大顶堆，按出现频率，也就是HashMap第二个元素排序
        priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(cmp)> pq(cmp);
        for (auto p: mp) {
            pq.push(p);
        }
        vector<int> res;
        // 弹出前k个元素
        while (k-- && !pq.empty()) {
            auto [x, y] = pq.top();
            pq.pop();
            res.push_back(x);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();
        // 字典记下所有数字出现频率
        for (int num: nums) {
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        // 降序排列比较函数
        // 大顶堆，按出现频率，也就是HashMap第二个元素排序
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[1] - a[1]);
        for (Map.Entry<Integer, Integer> entry: map.entrySet()) {
            int num = entry.getKey();
            int fre = entry.getValue();
            pq.add(new int[]{num, fre});
        }
        int[] res = new int[k];
        // 弹出前k个元素
        for (int i = 0; i < k && !pq.isEmpty(); ++i) {
            res[i] = pq.poll()[0];
        }
        return res;
    }
}
```

### 061 和最小的k个数对

方法：维护一个大小为k的优先队列存放两个数组的下标，按数对和排序。由于两个数组是升序的，可以先以一个数组为基准放一部分进优先队列，然后以另一个数组为基准“边取边放”，这样优化了空间同时也不会重复。如果两两比遍历都放进优先队列，会到时mn的双复杂度

注意，本题用两个指针从两个数组从头开始遍历的方式会漏

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
        // 以两个数组下标的数对和作为比较
        auto cmp = [&nums1, &nums2](const pair<int, int>& a, const pair<int, int>& b) {
            return nums1[a.first] + nums2[a.second] > nums1[b.first] + nums2[b.second];
        };
        int m = nums1.size();
        int n = nums2.size();
        vector<vector<int>> res;
        priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);
        // 先放一部分，然后在边放边取,放下标，可以去除重复枚举
        // (0, 0), (1, 0), (2, 0)..
        for (int i = 0; i < min(k, m); ++i) {
            pq.emplace(i, 0);
        }
        // 取k个最小数对
        while (k-- > 0 && !pq.empty()) {
            // 先取出目前优先队列中最小数对下标
            auto [x, y] = pq.top();
            pq.pop();
            res.push_back({nums1[x], nums2[y]});
            // 放入可能的最小数对下标
            // (0, 1) (1, 1), (2, 1)...
            if (y + 1 < n) {
                pq.emplace(x, y+1);
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        // 以两个数组下标的数对和作为比较
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b)->
            nums1[a[0]] + nums2[a[1]] - nums1[b[0]] - nums2[b[1]]
        );
        int m = nums1.length;
        int n = nums2.length;
        // 先放一部分，然后在边放边取,放下标，可以去除重复枚举
        // (0, 0), (1, 0), (2, 0)..
        for (int i = 0; i < Math.min(k, m); ++i) {
            pq.offer(new int[]{i, 0});
        }
        List<List<Integer>> res = new ArrayList<>();
        // 取k个最小数对
        while (k-- > 0 && !pq.isEmpty()) {
            // 先取出目前优先队列中最小数对下标
            int[] idxs = pq.poll();
            List<Integer> list = new ArrayList<>();
            list.add(nums1[idxs[0]]);
            list.add(nums2[idxs[1]]);
            res.add(list);
            // 放入可能的最小数对下标
            // (0, 1) (1, 1), (2, 1)...
            if (idxs[1] + 1 < n) {
                pq.offer(new int[]{idxs[0], idxs[1]+1});
            }
        }
        return res;
    }
}
```

## 前缀数

[前缀树Trie](https://leetcode.cn/problems/implement-trie-prefix-tree/solution/trie-tree-de-shi-xian-gua-he-chu-xue-zhe-by-huwt/)值一棵多叉树模型，每个结点没有保存字符值的数据成员，而是通过映射，在指向孩子结点的指针中保存类当前节点而言的下一个可能出现的左右字符的连接。

前缀树的基本实现及其方法如 062 实现前缀树所示

对于一些题，前缀树节点中还需要保护一些其他信息，孩子结点也不一定是字母的的映射，孩子结点映射可以是HashMap或数组，需要变通

### 062 实现前缀树所示

方法：最基本的前缀树实现

C++代码：

```C++
class Trie {
public:
    vector<Trie*> children;
    bool isEnd;
    /** Initialize your data structure here. */
    Trie(): isEnd(false) {
        children.resize(26);
    }
    
    /** Inserts a word into the trie. */
    void insert(string word) {
        Trie* node = this;
        for (char& c: word) {
            if (node->children[c-'a'] == nullptr) {
                node->children[c-'a'] = new Trie();
            }
            node = node->children[c-'a'];
        }
        node->isEnd = true;
    }
    
    /** Returns if the word is in the trie. */
    bool search(string word) {
        Trie* node = this;
        for (char& c: word) {
            if (node->children[c-'a'] == nullptr) {
                return false;
            }
            node = node->children[c-'a'];
        }
        return node->isEnd;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    bool startsWith(string prefix) {
        Trie* node = this;
        for (char& c: prefix) {
            if (node->children[c-'a'] == nullptr) {
                return false;
            }
            node = node->children[c-'a'];
        }
        return true;
    }
};
```

Java代码

```Java
class Trie {
    private Trie[] children;
    private boolean isEnd;
    /** Initialize your data structure here. */
    public Trie() {
        children = new Trie[26];
        isEnd = false;
    }
    
    /** Inserts a word into the trie. */
    public void insert(String word) {
        Trie node = this;
        for (int i = 0; i < word.length(); ++i) {
            char ch = word.charAt(i);
            if (node.children[ch - 'a'] == null) {
                node.children[ch - 'a'] = new Trie();
            }
            node = node.children[ch - 'a'];
        }
        node.isEnd = true;
    }
    
    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        Trie node = this;
        for (int i = 0; i < word.length(); ++i) {
            char ch = word.charAt(i);
            if (node.children[ch - 'a'] == null) {
                return false;
            }
            node = node.children[ch - 'a'];
        }
        return node.isEnd;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
        Trie node = this;
        for (int i = 0; i < prefix.length(); ++i) {
            char ch = prefix.charAt(i);
            if (node.children[ch - 'a'] == null) {
                return false;
            }
            node = node.children[ch - 'a'];
        }
        return true;
    }
}
```

### 063 替换单词

方法：使用字典树记录词根，然后在字典树中查找是否存在句子每个单词的词根

C++代码：

```C++
struct Trie {
    unordered_map<char, Trie*> children;
    bool isEnd = false;
};

class Solution {
public:
    string replaceWords(vector<string>& dictionary, string sentence) {
        // 新建字典树并将字典插入其中
        Trie* root = new Trie();
        for (string& s: dictionary) {
            insert(s, root);
        }
        vector<string> words = split(sentence);
        string res;
        // 对于每个单词，查找词根，拼接答案句子
        for (string& word: words) {
            res += findRoot(word, root);
            res += " ";
        }
        res.pop_back();
        return res;
    }

    // 字典树的插入
    void insert(string& word, Trie* root) {
        Trie* node = root;
        for (char& c: word) {
            if (!node->children.count(c)) {
                node->children[c] = new Trie();
            }
            node = node->children[c];
        }
        node->isEnd = true;
    }

    // 将句子中的单词以空格分隔为一个字符串数组
    vector<string> split(string& str) {
        int pos = 0;
        int start = 0;
        vector<string> ret;
        while (pos < str.size()) {
            while (pos < str.size() && str[pos] == ' ') {
                ++pos;
            }
            start = pos;
            while (pos < str.size() && str[pos] != ' ') {
                ++pos;
            }
            if (start < str.size()) {
                ret.emplace_back(str.substr(start, pos - start));
            }
        }
        return ret;
    }

    // 找到词根
    string findRoot(string& word, Trie *trie) {
        string root;
        for (char& c: word) {
            // 到达一个词根结尾，返回这个词根
            if (trie->isEnd) {
                return root;
            }
            // 不存在这个单词的词根，返回这个单词
            if (!trie->children.count(c)) {
                return word;
            }
            // 往下查找存在的词根
            root.push_back(c);
            trie = trie->children[c];
        }
        return root;
    }
};
```

Java代码

```Java
class Trie {
    Trie[] children = new Trie[26];
    boolean isEnd = false;
}

class Solution {
    private Trie root = new Trie(); // 新建字典树
    public String replaceWords(List<String> dictionary, String sentence) {
        // 将字典插入其中
        for (String word: dictionary) {
            insert(word);
        }
        String[] words = sentence.split(" ");
        // 对于每个单词，查找词根, 替换原来单词
        for (int i = 0; i < words.length; ++i) {
            words[i] = findRoot(words[i]);
        }
        return String.join(" ", words);
    }

    // 字典树的插入
    public void insert(String word) {
        Trie node = root;
        for (int i = 0; i < word.length(); ++i) {
            char c = word.charAt(i);
            if (node.children[c - 'a'] == null) {
                node.children[c - 'a'] = new Trie();
            }
            node = node.children[c - 'a'];
        }
        node.isEnd = true;
    }

    // 找到一个单词的词根
    String findRoot(String word) {
        Trie node = root;
        for (int i = 0; i < word.length(); ++i) {
            int c = word.charAt(i);
            // 到达一个词根结尾，返回这个词根
            if (node.isEnd) {
                return word.substring(0, i);
            }
            // 不存在这个单词的词根，返回这个单词
            if (node.children[c - 'a'] == null) {
                return word;
            }
            // 往下查找存在的词根
            node = node.children[c - 'a'];
        }
        return word;
    }
}
```

### 064 神奇的字典

方法：使用单词列表构建字典树，然后dfs，查找是否存在要求单词。在dfs向下搜索过程中，如果存在相同前缀，则继续向下遍历，如果出现不同的前缀，则标记其为修改，继续遍历。如果标记了为修改后又出现了不同，则放回false。

C++代码：

```C++
class Trie {
public:
    vector<Trie*> children;
    bool isEnd;
    Trie(): isEnd(false) {
        children.resize(26);
    }
};

class MagicDictionary {
public:
    /** Initialize your data structure here. */
    Trie* root;
    MagicDictionary() {
        root = new Trie();
    }
    
    void buildDict(vector<string> dictionary) {
        for (string& s: dictionary) {
            insert(s);
        }
    }
    
    bool search(string searchWord) {
        return dfs(searchWord, root, 0, false);
    }

    bool dfs(string& s, Trie* node, int idx, bool modified) {
        // 答案要求修改过一次，且长度到结尾
        if (idx == s.size()) {
            return modified && node->isEnd;
        }
        char c = s[idx];
        // 拥有相同前缀，向下遍历
        if (node->children[c-'a'] != nullptr) {
            if (dfs(s, node->children[c-'a'], idx+1, modified)) {
                return true;
            }
        }
        // 出现了不同单词
        if (!modified) {
            // 只出现过一个不同单词
            for (int i = 0; i < 26; ++i) {
                // 找到同级中所有可以替换的字母，进行遍历
                if (i != c-'a' && node->children[i] != nullptr) {
                    if (dfs(s, node->children[i], idx+1, true)) {
                        return true;
                    }
                }
            }
        }
        // 两个if都没通过，说明出现过两个不同单词
        return false;
    }

    void insert(string& s) {
        Trie* cur = this->root;
        for (char& c: s) {
            if (cur->children[c-'a'] == nullptr) {
                cur->children[c-'a'] = new Trie();
            }
            cur = cur->children[c-'a'];
        }
        cur->isEnd = true;
    }
};
```

Java代码

```Java
class Trie {
    boolean isEnd;
    Trie[] children;

    public Trie() {
        isEnd = false;
        children = new Trie[26];
    }
}

class MagicDictionary {
    Trie root;
    /** Initialize your data structure here. */
    public MagicDictionary() {
        root = new Trie();
    }

    public void buildDict(String[] dictionary) {
        for (String word: dictionary) {
            insert(word);
        }
    }
    
    public boolean search(String searchWord) {
        return dfs(searchWord, root, 0, false);
    }

    private void insert(String s) {
        Trie cur = root;
        for (int i = 0; i < s.length(); ++i) {
            char c = s.charAt(i);
            if (cur.children[c - 'a'] == null) {
                cur.children[c - 'a'] = new Trie();
            }
            cur = cur.children[c - 'a'];
        }
        cur.isEnd = true;
    }

    private boolean dfs(String s, Trie node, int idx, boolean modified) {
        // 答案要求修改过一次，且长度到结尾
        if (idx == s.length()) {
            return modified && node.isEnd;
        }
        char c = s.charAt(idx);
        // 拥有相同前缀，向下遍历
        if (node.children[c - 'a'] != null) {
            if (dfs(s, node.children[c - 'a'], idx + 1, modified)) {
                return true;
            }
        }
        // 出现了不同单词
        if (!modified) {
            // 只出现过一个不同单词
            for (int i = 0; i < 26; ++i) {
                // 找到同级中所有可以替换的字母，进行遍历
                if (i != c - 'a' && node.children[i] != null) {
                    if (dfs(s, node.children[i], idx + 1, true)) {
                        return true;
                    }
                }
            }
        }
        // 两个if都没通过，说明出现过两个不同单词
        return false;
    }
}
```

### 065 最短的单词编码

方法：

C++代码：用所有单词的逆序构建字典树，字典树每个节点记下当前树的深度。在每次插入时，用HashMap记下叶子节点以及此次插入的单词在数组中的索引。并有对每个结点用一个属性记下该字母作为前缀出现的次数

```C++
struct Trie {
    vector<Trie*> children;
    int count;  // 用于记录该节点字母作为前缀出现次数
    Trie():children(26, nullptr),count(0) {
        
    }
};
class Solution {
public:
    int minimumLengthEncoding(vector<string>& words) {
        unordered_map<Trie*, int> mp;
        Trie* root = new Trie();
        // 构建字典树，并用HashMap记下叶子节点以及此次插入的单词在数组中的索引
        for (int i = 0; i < words.size(); ++i) {
            Trie* node = insert(root, words[i]);
            mp[node] = i;
        }
        int res = 0;
        for (auto &[node, i]: mp) {
            // count == 0 说明此单词不是任何一个单词的前缀
            // 否则，count不等于0说明此单词是某个单词前缀，不计入答案
            if (node->count == 0) {
                res += words[i].size() + 1; // 单词长度再加上一个#
            }
        }
        return res;
    }

    // 插入后需要用HashMap记录，所以需要返回最后一个节点
    Trie* insert(Trie* node, string& s) {
        for (int i = s.size()-1; i >= 0; --i) {
            int idx = s[i] - 'a';
            if (node->children[idx] == nullptr) {
                node->children[idx] = new Trie();
                ++node->count;  // 当前结点作为前缀的出现次数+1
            }
            node = node->children[idx];
        }
        return node;    // 返回单词最后一个节点
    }
};
```

Java代码

```Java
class Trie {
    Trie[] children;
    int count;
    Trie() {
        children = new Trie[26];
        count = 0;
    }
}

class Solution {
    public int minimumLengthEncoding(String[] words) {
        Map<Trie, Integer> map = new HashMap<>();
        Trie root = new Trie();
        // 构建字典树，并用HashMap记下叶子节点以及此次插入的单词在数组中的索引
        for (int i = 0; i < words.length; ++i) {
            Trie node = insert(root, words[i]);
            map.put(node, i);
        }
        int res = 0;
        for (Map.Entry<Trie, Integer> entry: map.entrySet()) {
            // count == 0 说明此单词不是任何一个单词的前缀
            // 否则，count不等于0说明此单词是某个单词前缀，不计入答案
            Trie node = entry.getKey();
            int i = entry.getValue();
            if (node.count == 0) {
                res += words[i].length() + 1;// 单词长度再加上一个#
            }
        }
        return res;
    }

    // 插入后需要用HashMap记录，所以需要返回最后一个节点
    Trie insert(Trie node, String s) {
        for (int i = s.length() - 1; i >= 0; --i) {
            int idx = s.charAt(i) - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new Trie();
                ++node.count;
            }
            node = node.children[idx];
        }
        return node;    // 返回单词最后一个节点
    }
}
```

### 066 单词之和

方法：使用前缀树存储单词，在一个词的结尾记下该单词对应的值，获取以该前缀开头的键时，使用dfs遍历所有分支计算总和

C++代码：

```C++
struct Trie {
    vector<Trie*> children;
    int val;
    Trie():children(26, nullptr),val(0) {

    }
};

class MapSum {
public:
    /** Initialize your data structure here. */
    Trie* root;
    MapSum() {
        root = new Trie();
    }
    
    void insert(string key, int val) {
        Trie* cur = this->root;
        for (char c: key) {
            int idx = c-'a';
            if (cur->children[idx] == nullptr) {
                cur->children[idx] = new Trie();
            }
            cur = cur->children[idx];
        }
        cur->val = val;
    }
    
    int sum(string prefix) {
        // 遍历到此前缀的结尾
        Trie* cur = this->root;
        for (char c: prefix) {
            int idx = c-'a';
            // 没有此前缀的单词，直接返回
            if (cur->children[idx] == nullptr) {
                return 0;
            }
            cur = cur->children[idx];
        }
        // dfs遍历所有以此为前缀的单词的和
        return dfsSearch(cur);
    }

    int dfsSearch(Trie* node) {
        if (node == nullptr) {
            return 0;
        }
        int sum = node->val;
        // 对于每一个分支，进行遍历
        for (int i = 0; i < 26; ++i) {
            if (node->children[i] != nullptr) {
                sum += dfsSearch(node->children[i]);
            }
        }
        return sum;
    }
};
```

Java代码

```Java
class Trie {
    Trie[] children;
    int val;
    Trie() {
        children = new Trie[26];
        val = 0;
    }
}

class MapSum {
    private Trie root;
    /** Initialize your data structure here. */
    public MapSum() {
        root = new Trie();
    }
    
    public void insert(String key, int val) {
        Trie cur = root;
        for (int i = 0; i < key.length(); ++i) {
            int idx = key.charAt(i) - 'a';
            if (cur.children[idx] == null) {
                cur.children[idx] = new Trie();
            }
            cur = cur.children[idx];
        }
        cur.val = val;
    }
    
    public int sum(String prefix) {
        // 遍历到此前缀的结尾
        Trie cur = root;
        for (int i = 0; i < prefix.length(); ++i) {
            int idx = prefix.charAt(i) - 'a';
            // 没有此前缀的单词，直接返回
            if (cur.children[idx] == null) {
                return 0;
            }
            cur = cur.children[idx];
        }
        // dfs遍历所有以此为前缀的单词的和
        return dfsSearch(cur);
    }

    int dfsSearch(Trie node) {
        if (node == null) {
            return 0;
        }
        int sum = node.val;
        // 对于每一个分支，进行遍历
        for (int i = 0; i < 26; ++i) {
            if (node.children != null) {
                sum += dfsSearch(node.children[i]);
            }
        }
        return sum;
    }
}
```

### 067 最大的异或

方法：用数组中每个数的二进制表示，构建只有两个分叉的字典树。然后在查询过程中，每次“尽量”向当前位的另一边移动，这样异或为1（由于是从高位到低位，所以尽量保证高位为1），最后的值最大

C++代码：

```C++
struct Trie {
    Trie* left = nullptr;   // 左边表示0
    Trie* right = nullptr;  // 右边表示1
    Trie() {}
};

class Solution {
public:
    Trie* root = new Trie();
    int findMaximumXOR(vector<int>& nums) {
        int n = nums.size();
        int x = 0;
        for (int i = 1; i < n; ++i) {
            // 每次插入后再进行异或比较，能包含所有可能异或运算
            insert(nums[i-1]);
            x = max(x, check(nums[i]));
        }
        return x;
    }

    void insert(int num) {
        Trie* cur = root;
        // 每个数不超过2^30
        for (int k = 30; k >= 0; --k) {
            // 取第k位是0还是1
            int bit = (num >> k) & 1;
            if (bit == 0) {
                // 左边表示0
                if (cur->left == nullptr) {
                    cur->left = new Trie();
                }
                cur = cur->left;
            } else {
                // 右边表示1
                if (cur->right == nullptr) {
                    cur->right = new Trie();
                }
                cur = cur->right;
            }
        }
    }

    int check(int num) {
        Trie* cur = root;
        int x = 0;
        for (int k = 30; k >= 0; --k) {
            int bit = (num >> k) & 1;
            // 尽量往不同于当前位的方向走，这样异或出来最大
            if (bit == 0) {
                // 当前位为0，向右边走
                if (cur->right) {
                    cur = cur->right;
                    x = x * 2 + 1;  // 和当前位不同，异或为1，左移一位加上1
                } else {
                    cur = cur->left;
                    x = x * 2;
                }
            } else {
                // 当前位为1，向左边走
                if (cur->left) {
                    cur = cur->left;
                    x = x * 2 + 1;
                } else {
                    cur = cur->right;
                    x = x * 2;
                }
            }
        }
        return x;
    }
};
```

Java代码

```Java
class Trie {
    Trie left = null; // 左子树指向表示 0 的子节点
    Trie right = null; // 右子树指向表示 1 的子节点
}

class Solution {
    private Trie root = new Trie();
    public int findMaximumXOR(int[] nums) {
        int n = nums.length;
        int res = 0;
        for (int i = 1; i < n; ++i) {
            // 每次插入后再进行异或比较，能包含所有可能异或运算
            insert(nums[i-1]);
            res = Math.max(res, check(nums[i]));
        }
        return res;
    }

    private void insert(int num) {
        Trie cur = root;
        // 每个数不超过2^30
        for (int i = 30; i >= 0; --i) {
            // 取第k位是0还是1
            int bit = (num >> i) & 1;
            if (bit == 0) {
                // 左边表示0
                if (cur.left == null) {
                    cur.left = new Trie();
                }
                cur = cur.left;
            } else {
                // 右边表示1
                if (cur.right == null) {
                    cur.right = new Trie();
                }
                cur = cur.right;
            }
        }
    }

    private int check(int num) {
        Trie cur = root;
        int x = 0;
        for (int i = 30; i >= 0; --i) {
            int bit = (num >> i) & 1;
            // 尽量往不同于当前位的方向走，这样异或出来最大
            if (bit == 0) {
                if (cur.right != null) {
                    cur = cur.right;
                    x = (x << 1) + 1; // 和当前位不同，异或为1，左移一位加上1
                } else {
                    cur = cur.left;
                    x = x << 1;
                }
            } else {
                // 当前位为1，向左边走
                if (cur.left != null) {
                    cur = cur.left;
                    x = (x << 1) + 1;
                } else {
                    cur = cur.right;
                    x = x << 1;
                }
            }
        }
        return x;
    }
}
```

## 二分查找

* 循环可以继续的条件left < right，退出循环以后，left和right重合
* 区间划分两种情况1：[left...mid]和[mid+1...right]; 2：[left..mid-1]和[mid...right]，需注意注意事项2

二分查找的问题关键在于分析清楚什么情况下搜索区间是什么
需要清楚：
循环可以继续的条件
mid位置的值是否可能是问题的答案
下一轮搜索区间向左还是向右

注意事项：

1. 只有把区间分层两部分，配合while(left < right)，退出循环以后才有left与right重合
2. 看到left = mid和right = mid - 1，需要将取中间数改成上取整，这样避免只有两个数时，left一直等于mid导致的死循环
3. 如果答案可能不存在，退出循环后需要再做一次判断
4. right = n 和 right = n-1是不固定的，因为n也可能是要求答案，需要变通

### 068 查找插入位置

方法：二分查找，注意末尾后一个位置也可能是插入位置，要查找插入位置，右区间移动时，要包含mid，即可能的插入位置

C++代码：

```C++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int n = nums.size();
        int left = 0;
        int right = n-1;
        while (left < right) {
            int mid = (left + right) >> 1;
            // 找到目标值
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] > target) {
                // 因为要查找插入位置的下标，在这种情况下，mid也可能是插入值的位置
                right = mid;
            } else {
                // 此时target一定不会在mid插入，要插入也是在mid下一个位置
                left = mid + 1;
            }
        }
        return left;
    }
};
```

Java代码

```Java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int n = nums.length;
        int left = 0;
        int right = n;
        while (left < right) {
            int mid = (left + right) / 2;
            if (nums[mid] < target) {
                // 此时target一定不会在mid插入，要插入也是在mid下一个位置
                left = mid + 1;
            } else {
                // 因为要查找插入位置的下标，在这种大于target的情况下，mid也可能是插入值的位置
                right = mid;
            }
        }
        return left;
    }
}
```

### 069 山峰数组的顶点

方法：使用二分查找，利用中间点和中间的右边的点的大小关系（单调性）来判断此时位于山峰的左侧还是右侧

C++代码：

```C++
class Solution {
public:
    int peakIndexInMountainArray(vector<int>& arr) {
        int n = arr.size();
        // 首尾两个位置不可能是山峰
        int left = 1;
        int right = n - 2;
        while (left < right) {
            int mid = (left + right) / 2;
            // 利用中间点和中间的右边的点的大小关系（单调性）来判断此时位于山峰的左侧还是右侧
            if (arr[mid] > arr[mid+1]) {
                // mid可能是山峰，需要包含进区间
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return right;
    }
};
```

Java代码

```Java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        // 首尾两个位置不可能是山峰
        int left = 1;
        int right = arr.length - 2;
        while (left < right) {
            int mid = (left + right) / 2;
            // 利用中间点和中间的右边的点的大小关系（单调性）来判断此时位于山峰的左侧还是右侧
            if (arr[mid] < arr[mid+1]) {
                // 此时mid一定不是山峰
                left = mid + 1;
            } else {
                // 此时mid可能是山峰，需要包含进区间
                right = mid;
            }
        }
        return left;
    }
}
```

### 070 排序数组中只出现一次的数字

方法：如果数字两两出现的话，下标为偶数的数和它右边一位相等，如果出现类不相等，说明之前混入了只出现一次的数字，可以一次来二分，看只出现一次的数字在左边还是右边

C++代码：

```C++
class Solution {
public:
    int singleNonDuplicate(vector<int>& nums) {
        int left = 0;
        int right = nums.size()-1;
        while (left < right) {
            int mid = (left + right) / 2;
            // 需要考虑mid的奇偶性
            // 若只出现一次数字在右边，则mid为偶数和右边一个数比
            if (mid % 2 == 0) {
                if (nums[mid] == nums[mid+1]) {
                    left = mid + 1;
                } else {
                    right = mid;
                }
            } else {
                // mid为奇数和左边一个数比
                if (nums[mid] == nums[mid-1]) {
                    left = mid + 1;
                } else {
                    right = mid;
                }
            }
        }
        return nums[left];
    }
};
```

Java代码

```Java
class Solution {
    public int singleNonDuplicate(int[] nums) {
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            int mid = (left + right) / 2;
            // 需要考虑mid的奇偶性
            // 若只出现一次数字在右边，则mid为偶数和右边一个数比,mid为奇数和左边一个数比
            // 这里使用mid^1是个技巧, 当mid为偶数时，mid^1 = mid+1,当mid奇数，mid^1 = mid-1
            if (nums[mid] == nums[mid^1]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return nums[left];
    }
}
```

### 071 按权重生成随机数

方法：先用一个数组记下数组的前缀和，然后随机生成一个大小在0到数组总和之间的一个数，用二分查找找到这个数位于前缀和数组的那个位置，这个位置就是选取符合概率要求的数组下标。

C++代码：

```C++
class Solution {
public:
    vector<int> preSum;
    Solution(vector<int>& w) {
        // 累加求前缀和
        partial_sum(w.begin(), w.end(), back_inserter(preSum));
    }
    
    int pickIndex() {
        // 随机生成一个大小在0到数组总和之间的一个数
        int r = rand() % preSum.back();
        // 返回当前迭代器到开始迭代器的距离，也就是数组的下标
        return upper_bound(preSum.begin(), preSum.end(), r) - preSum.begin();
    }
};
```

Java代码

```Java
class Solution {
    int sum[];
    public Solution(int[] w) {
        int n = w.length;
        // 为了循环内方便运算，前缀多开一个空间，从1到n，下标0空置
        sum = new int[n+1];
        for (int i = 1; i <= n; ++i) {
            sum[i] += sum[i-1] + w[i-1];
        }
    }
    
    public int pickIndex() {
        int n = sum.length;
        // 随机生成一个大小在0到数组总和之间的一个数
        int t = (int)(Math.random() * sum[n-1]) + 1;
        int left = 1;
        int right = n - 1;
        // 二分查找
        while (left < right) {
            int mid = left + right >> 1;
            if (sum[mid] >= t) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        //前缀和数组和原数组下标差1
        return right - 1;
    }
}
```

### 072 求平方根

方法：采用二分查找，从1到x/2中找到一个最大的数mid使mid * mid <=
x。
[牛顿法](https://leetcode.cn/problems/jJ0w9p/solution/qiu-ping-fang-gen-by-leetcode-solution-ybnw/)

C++代码：

```C++
class Solution {
public:
    int mySqrt(int x) {
        // 牛顿法迭代
        if (x == 0) {
            return 0;
        }
        double C = x;
        double x0 = x;
        while (true) {
            // 迭代公式
            double xi = 0.5 * (x0 + C / x0);
            // 达到该精度即可完成计算
            if (fabs(x0 - xi) < 1e-7) {
                break;
            }
            x0 = xi;
        }
        return int(x0);
    }
};
```

Java代码

```Java
class Solution {
    public int mySqrt(int x) {
        // 0 和 1 的平方根就是自己
        if (x == 0 || x == 1) {
            return x;
        }
        int left = 1;
        int right = x / 2;
        // 二分查找
        while (left < right) {
            // 注意此时向上取整
            int mid = (right - left + 1) / 2 + left;
            if (mid > x / mid) {
                right = mid - 1;
            } else {
                // 因为保留整数部分，意味着mid^2是小于等于x的最大数
                left = mid;
            }
        }
        return left;
    }
}
```

### 073 狒狒吃香蕉

方法：采用二分查找搜索吃完时间小于等于h的速度，速度范围为每小时1根到每小时吃最多一堆的数量。

C++代码：

```C++
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int h) {
        int left = 1;
        int right = 0;
        // 找到最快速度作为搜索区间
        for (int& x: piles) {
            right = max(x, right);
        }
        while (left < right) {
            int mid = (right - left) / 2 + left;
            // 计算吃的时间
            long time = eatTime(piles, mid);
            if (time > h) {
                // 下一次的搜索区间不包括mid，因为time只能小于等于h而不能大于h
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    // 计算以速度k吃完需要的时间
    long eatTime(vector<int>&piles, int k) {
        long sum = 0;
        for (int x: piles) {
            if (x % k == 0) {
                sum += x / k;
            } else {
                sum += x / k + 1;
            }
        }
        return sum;
    }
};
```

Java代码

```Java
class Solution {
    public int minEatingSpeed(int[] piles, int h) {
        int left = 1;
        int right = 0;
        // 找到最快速度作为搜索区间
        for (int x: piles) {
            right = Math.max(x, right);
        }
        while (left < right) {
            int mid = (right - left) / 2 + left;
            // 计算吃的时间
            long time = eatTime(piles, mid);
            if (time > h) {
                // 下一次的搜索区间不包括mid，因为time只能小于等于h而不能大于h
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    // 计算以速度k吃完需要的时间
    private long eatTime(int[] piles, int k) {
        long sum = 0;
        for (int x: piles) {
            if (x % k == 0) {
                sum += x / k;
            } else {
                sum += x / k + 1;
            }
        }
        return sum;
    }
}
```

## 排序

### 074 合并区间

方法：对区间使用先按第一个数升序，再按第二个数降序的排序方式。然后遍历排序好的数组，先确定左边界，然后找到重叠的最大右边界，当最大右边界小于下一个区间的左边界，进入下一个循环

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        // 对区间使用先按第一个数升序，再按第二个数降序的排序方式
        sort(intervals.begin(), intervals.end(), [](auto& v1, auto& v2) {
            if (v1[0] < v2[0]) {
                return true;
            } else if (v1[0] == v2[0]){
                return v1[1] > v2[1];
            } else {
                return false;
            }
        });

        vector<vector<int>> res;
        int n = intervals.size();
        int i = 0;
        while (i < n) {
            // 先确定左边界
            int a = intervals[i][0];
            int b = intervals[i][1];
            ++i;
            // 找到小于下一个区间的左边界的最大右边界
            while (i < n && b >= intervals[i][0]) {
                b = max(b, intervals[i][1]);
                ++i;
            }
            // 此时确定了一个区间，放进答案，进入下一个循环
            res.push_back({a, b});
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int[][] merge(int[][] intervals) {
        // 对区间使用先按第一个数升序的排序方式
        Arrays.sort(intervals, (v1, v2) -> v1[0] - v2[0]);
        List<int[]> merged = new ArrayList<>();
        int n = intervals.length;
        int i = 0;
        while (i < n) {
            // 先确定左边界
            int a = intervals[i][0];
            int b = intervals[i][1];
            ++i;
            // 找到小于下一个区间的左边界的最大右边界
            while (i < n && b >= intervals[i][0]) {
                b = Math.max(b, intervals[i][1]);
                ++i;
            }
            // 此时确定了一个区间，放进答案，进入下一个循环
            merged.add(new int[]{a, b});
        }
        // 返回答案要求格式
        return merged.toArray(new int[merged.size()][]);
    }
}
```

### 075 数组相对排序

方法：用哈希表先记下arr2中的相对位置，key为值，value为相对位置（即下标）。然后自定义排序规则，当arr1两个数都出现在了arr2，则按相对位置排，如果只有一个数出现，将这个数放在前面，否则，按数值升序排。

C++代码：

```C++
class Solution {
public:
    vector<int> relativeSortArray(vector<int>& arr1, vector<int>& arr2) {
        // 先用哈希表记下相对位置
        unordered_map<int, int> mp;
        for (int i = 0; i < arr2.size(); ++i) {
            mp[arr2[i]] = i+1;
        }
        // 自定义排序规则
        sort(arr1.begin(), arr1.end(), [&mp](int a, int b) {
            if (mp.count(a) && mp.count(b)) {
                // 两个数都在arr2，按相对位置排
                return mp[a] < mp[b];
            } else if (mp.count(a)) {
                // 只有a出现，a放前面
                return true;
            } else if (mp.count(b)) {
                // 只有b出现，b放前面
                return false;
            } else {
                // 都没出现，按升序排
                return a < b;
            }
        });
        return arr1;
    }
};
```

Java代码

```Java
class Solution {
    public int[] relativeSortArray(int[] arr1, int[] arr2) {
        // 先用哈希表记下相对位置
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < arr2.length; ++i) {
            map.put(arr2[i], i);
        }
        List<Integer> list = new ArrayList<>();
        for (int x: arr1) {
            list.add(x);
        }
        // 自定义排序规则
        Collections.sort(list, (a, b) -> {
            if (map.containsKey(a) && map.containsKey(b)) {
                // 两个数都在arr2，按相对位置排
                return map.get(a) - map.get(b);
            } else if (map.containsKey(a)) {
                // 只有a出现，a放前面
                return -1;
            } else if (map.containsKey(b)) {
                // 只有b出现，b放前面
                return 1;
            }
            // 都没出现，按升序排
            return a - b;
        });
        for (int i = 0; i < list.size(); ++i) {
            arr1[i] = list.get(i);
        }
        return arr1;
    }
}
```

### 076 数组中的第k大的数字

方法：采用类似快速排序的方式，应为这种方式每次都能将一个数放到排序数组中的正确位置，找到以下标k作为轴的位置即可停止，否则，朝

C++代码：

```C++
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        quickPartition(nums, 0, nums.size()-1, k-1);
        // 若按降序排列，第k大的数就是下标为k-1的数
        return nums[k-1];
    }

    // 类似快排的划分，每次都能选择一个数放到在排序的正确位置。
    void quickPartition(vector<int>& nums, int left, int right, int target) {
        // 随机在left到right之间选一个位置的值作为轴
        int random = rand() % (right - left + 1) + left;
        // 将轴记录到最左边的位置，用一个数记下选定的值
        int base = nums[random];
        swap(nums[left], nums[random]);
        int index = left;
        // 将大于轴的数字移动到左边，最后轴左边都是比它大的数，index在轴的位置
        for (int i = left + 1; i <= right; ++i) {
            if (nums[i] >= base) {
                // 从
                swap(nums[++index], nums[i]);
            }
        }
        // 将当前轴的位置上的值换回
        swap(nums[index], nums[left]);
        // 如果当前的轴在目标右边，查找左边部分
        if (index > target) {
            quickPartition(nums, left, index-1, target);
        } else if (index < target) {
            // 如果当前的轴在目标左边，查找右边部分
            quickPartition(nums, index+1, right, target);
        }
        // 如果index==target，即找到目标位置，即可不再进行排序。
    }
};
```

Java代码

```Java
class Solution {
    Random random = new Random();
    public int findKthLargest(int[] nums, int k) {
        quickPartition(nums, 0, nums.length-1, k-1);
        // 若按降序排列，第k大的数就是下标为k-1的数
        return nums[k-1];
    }

    // 类似快排的划分，每次都能选择一个数放到在排序的正确位置。
    public void quickPartition(int[] nums, int left, int right, int target) { 
        // 随机在left到right之间选一个位置的值作为轴
        int randInt = random.nextInt(right - left + 1) + left;
        // 将轴记录到最左边的位置，用一个数记下选定的值
        int base = nums[randInt];
        swap(nums, left, randInt);
        int index = left;
        // 将大于轴的数字移动到左边，最后轴左边都是比它大的数，index在轴的位置
        for (int i = left + 1; i <= right; ++i) {
            if (nums[i] > base) {
                swap(nums, i, ++index);
                
            }
        }
        // 将当前轴的位置上的值换回
        swap(nums, index, left);
        if (index > target) {
            // 如果当前的轴在目标右边，查找左边部分
            quickPartition(nums, left, index-1, target);
        } else if (index < target) {
            // 如果当前的轴在目标左边，查找右边部分
            quickPartition(nums, index+1, right, target);
        }
        // 如果index==target，即找到目标位置，即可不再进行排序。
    }

    public void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```

### 077 链表排序

方法：使用归并排序的方法，将链表两两划分再进行归并

C++代码：

```C++
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        return sortList(head, nullptr);
    }

    // 将链表分成两部分，两两归并
    ListNode* sortList(ListNode* head, ListNode* tail) {    //tails是尾后一个节点
        if (head == nullptr) {
            return head;
        }
        if (head->next == tail) {
            // 此次归并只有一个结点
            head->next = nullptr;
            return head;
        }
        ListNode* fast = head;
        ListNode* slow = head;
        // 快慢指针找到中间结点
        while (fast != tail && fast->next != tail) {
            fast = fast->next->next;
            slow = slow->next;
        }
        // 递归进入过程先两两分开，在递归返回过程进行归并
        ListNode* list1 = sortList(head, slow);
        ListNode* list2 = sortList(slow, tail);
        return merge(list1, list2);
    }

    // 合并两个链表
    ListNode* merge(ListNode* node1, ListNode* node2) {
        ListNode* preNode = new ListNode();
        ListNode* cur = preNode;
        while (node1 != nullptr && node2 != nullptr) {
            if (node1->val < node2->val) {
                cur->next = node1;
                node1 = node1->next;
            } else {
                cur->next = node2;
                node2 = node2->next;
            }
            cur = cur->next;
        }
        if (node1 == nullptr) {
            cur->next = node2;
        }
        if (node2 == nullptr) {
            cur->next = node1;
        }
        return preNode->next;
    }
};
```

Java代码

```Java
class Solution {
    public ListNode sortList(ListNode head) {
        return sortList(head, null);
    }

    // 将链表分成两部分，两两归并
    //tails是尾后一个节点
    public ListNode sortList(ListNode head, ListNode tail) {
        if (head == null) {
            return head;
        }
        if (head.next == tail) {
            // 此次归并只有一个结点
            head.next = null;
            return head;
        }
        ListNode fast = head;
        ListNode slow = head;
        // 快慢指针找到中间结点
        while (fast != tail && fast.next != tail) {
            fast = fast.next.next;
            slow = slow.next;
        }
        // 递归进入过程先两两分开，在递归返回过程进行归并
        ListNode list1 = sortList(head, slow);
        ListNode list2 = sortList(slow, tail);
        return merge(list1, list2);
    }

    // 合并两个链表
    public ListNode merge(ListNode node1, ListNode node2) {
        ListNode preNode = new ListNode();
        ListNode cur = preNode;
        while (node1 != null && node2 != null) {
            if (node1.val < node2.val) {
                cur.next = node1;
                node1 = node1.next;
            } else {
                cur.next = node2;
                node2 = node2.next;
            }
            cur = cur.next;
        }
        if (node1 != null) {
            cur.next = node1;
        }
        if (node2 != null) {
            cur.next = node2;
        }
        return preNode.next;
    }
}
```

### 078 合并排序链表

方法：与排序链表类似，在数组中先进行链表的两两划分，再进行归并

C++代码：

```C++
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        return merge(lists, 0, lists.size()-1);
    }

    // 将链表数组两两划分，在两两归并
    ListNode* merge(vector<ListNode*>& lists, int left, int right) {
        if (left == right) {
            // 此时返回所指向的一个链表即可
            return lists[left];
        } else if (left > right) {
            return nullptr;
        }
        // 使用mid对链表数组进行划分
        int mid = (left + right) / 2;
        // 递归进入过程先两两分开，在递归返回过程进行归并
        ListNode* list1 = merge(lists, left, mid);
        ListNode* list2 = merge(lists, mid+1, right);
        // return mergeTwo(merge(lists, left, mid), merge(lists, mid+1, right));
        return mergeTwo(list1, list2);
    }

    // 归并两个链表
    ListNode* mergeTwo(ListNode* node1, ListNode* node2) {
        ListNode* preHead = new ListNode();
        ListNode* cur = preHead;
        while (node1 != nullptr && node2 != nullptr) {
            if (node1->val < node2->val) {
                cur->next = node1;
                node1 = node1->next;
            } else {
                cur->next = node2;
                node2 = node2->next;
            }
            cur = cur->next;
        }
        if (node1 != nullptr) {
            cur->next = node1;
        } else {
            cur->next = node2;
        }
        return preHead->next;
    }
};
```

Java代码

```Java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        return merge(lists, 0, lists.length-1);
    }

    // 将链表数组两两划分，在两两归并
    public ListNode merge(ListNode[] lists, int left, int right) {
        if (left == right) {
            // 此时返回所指向的一个链表即可
            return lists[left];
        } else if (left > right) {
            return null;
        }
        // 使用mid对链表数组进行划分
        int mid = (left + right) / 2;
        // 递归进入过程先两两分开，在递归返回过程进行归并
        ListNode list1 = merge(lists, left, mid);
        ListNode list2 = merge(lists, mid+1, right);
        // return mergeTwo(merge(lists, left, mid), merge(lists, mid+1, right));
        return mergeTwo(list1, list2);
    }

    // 归并两个链表
    public ListNode mergeTwo(ListNode node1, ListNode node2) {
        ListNode preHead = new ListNode();
        ListNode cur = preHead;
        while (node1 != null && node2 != null) {
            if (node1.val < node2.val) {
                cur.next = node1;
                node1 = node1.next;
            } else {
                cur.next = node2;
                node2 = node2.next;
            }
            cur = cur.next;
        }
        if (node1 != null) {
            cur.next = node1;
        } else {
            cur.next = node2;
        }
        return preHead.next;
    }
}
```

## 回溯法

当问题需要“回头”一次查找出所有解的时候，使用回溯算法。即满足结束条件或者发现不是正确路径的时候，要撤销选择，回退到上一个状态，继续尝试，直到找出所有解

[回溯算法解题步骤](https://leetcode.cn/problems/subsets/solution/c-zong-jie-liao-hui-su-wen-ti-lei-xing-dai-ni-gao-/)：

1. 画出递归树，找到状态变量（回溯函数的参数）*
2. 根据题意，确立结束条件
3. 找准选择列表（与函数参数相关），与第一步紧密关联*
4. 判断是否需要剪枝
5. 做出选择，递归调用，进入下一层
6. 撤销选择

回溯法的基本结构

```Java
public void dfs(选择列表, 路径) {
    if (..) {
        result.add();
        return;
    }
    for (选择: 选择列表) {
        if (visited.contains(做过选择)) {
            continue;
        }
        选择
        dfs(选择列表， 路径)
        撤销选择
    }
}

```

### 079 所有子集

方法：使用回溯法，穷举所有可能。可以将数组想像成一个二进制数，每个元素是取或不取

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<int> tmp;
        res.push_back({});  // 先加入一个空集
        dfs(nums, 0, tmp);
        return res;
    }

    void dfs(vector<int>& nums, int idx, vector<int> tmp) {
        if (idx == nums.size()) {
            // 访问到数组结尾时，递归结束
            return;
        }
        // 使用回溯法穷举，选择为取或不取
        dfs(nums, idx+1, tmp);
        tmp.push_back(nums[idx]);   // 回溯过程中的每一个子答案
        res.push_back(tmp); // 答案中加入穷举的结果
        dfs(nums, idx+1, tmp);  // 撤销选择
    }
};
```

Java代码

```Java
class Solution {
    ArrayList<Integer> tmp = new ArrayList<Integer>();
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    public List<List<Integer>> subsets(int[] nums) {
        dfs(nums, 0);
        return res;
    }

    public void dfs(int[] nums, int cur) {
        if (cur == nums.length) {
            // 遍历到结尾时，放入答案
            res.add(new ArrayList<Integer>(tmp));
            return ;
        }
        // 先递归到到底把所有元素当作子集，然后再进行放入、移除
        // 相当于是在递归返回时，进行的穷举
        tmp.add(nums[cur]);
        dfs(nums, cur+1);
        tmp.remove(tmp.size()-1);
        dfs(nums,cur+1);
    }
}
```

### 080 含有k个元素的组合

方法：回溯法穷举，对每个位置的数考虑取或不去

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;
    vector<vector<int>> combine(int n, int k) {
        dfs(1, n, k);
        return res;
    }

    void dfs(int cur, int n, int k) {
        if (tmp.size() + (n - cur + 1) < k) {
            // 剪枝，已有数组和剩下数字的数量不能组成k个数
            return ;
        }
        if (tmp.size() == k) {
            // 含有k个元素时，将路径加入答案，结束递归
            res.push_back(tmp);
            return ;
        }
        // 考虑当前位置
        tmp.push_back(cur);
        dfs(cur + 1, n, k);
        // 不考虑当前位置
        tmp.pop_back();
        dfs(cur + 1, n, k);
    }
};
```

Java代码

```Java
class Solution {
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    List<Integer> tmp = new ArrayList<Integer>();
    public List<List<Integer>> combine(int n, int k) {
        dfs(1, n, k);
        return res;
    }

    public void dfs(int cur, int n, int k) {
        // 剪枝，已有数组和剩下数字的数量不能组成k个数
        if (tmp.size() + (n - cur + 1) < k) {
            return ;
        }
        if (tmp.size() == k) {
            // 含有k个元素时，将路径加入答案，结束递归
            res.add(new ArrayList<Integer>(tmp));
            return;
        }
        // 考虑当前位置
        tmp.add(cur);
        dfs(cur+1, n, k);
        // 不考虑当前位置
        tmp.remove(tmp.size()-1);
        dfs(cur+1, n, k);
    }
}
```

### 081 允许重复选择元素的组合

方法：回溯算法加上剪枝，按从小到大逐个选择，即多次选择前面的数，再选择后面的数，这样就避免了遍历到后面又要重新选择前面。

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;    // 答案中的每一项，也作为回溯路径
    int curSum = 0;
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        dfs(candidates, 0,target);
        return res;
    }

    // begin是选择的起始位置，防止一个结果的多种组合放进答案
    void dfs(vector<int>& candidates,int begin, int target) {
        if (curSum == target) {
            // 路径上和为目标，加入答案
            res.push_back(tmp);
        }
        for (int i = begin; i < candidates.size(); ++i) {
            if (curSum > target) {
                break;
            }
            tmp.push_back(candidates[i]);
            curSum += candidates[i];
            dfs(candidates, i, target); // 从位置i开始回溯，不回头
            curSum -= candidates[i];
            tmp.pop_back();
        }
    }
};
```

Java代码

```Java
class Solution {
    // 答案中的每一项，也作为回溯路径
    ArrayList<Integer> tmp = new ArrayList<Integer>();
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        dfs(candidates, 0, target);
        return res;
    }

    public void dfs(int[] candidates, int cur, int  target) {
        if (target == 0) {
            // 路径上和为目标，加入答案并返回
            res.add(new ArrayList<Integer>(tmp));
            return;
        }
        if (cur >= candidates.length || target < 0) {
            return ;
        }
        // 选择下一个，不选当前的
        dfs(candidates, cur+1, target);
        if (target - candidates[cur] >= 0) {
            tmp.add(candidates[cur]);
            // 重复选择当前的
            dfs(candidates, cur, target - candidates[cur]);
            tmp.remove(tmp.size()-1);// 撤销选择     
        }
    }
}
```

### 082 含有重复元素集合的组合

方法：类似上一题，将数组先进行排序，再进行回溯选择，当元素重复则直接跳过

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp; // 答案中的每一项，也作为回溯路径
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        // 先进行排序
        sort(candidates.begin(), candidates.end());
        dfs(candidates, 0, target);
        return res;
    }

    void dfs(vector<int>& candidates, int cur, int target) {
        if (target == 0) {
            // 选取路径上和为目标，加入答案，结束递归
            res.push_back(tmp);
            return ;
        }
        for (int i = cur; i < candidates.size() && target - candidates[i] >= 0; ++i) {
            if (i > cur && candidates[i] == candidates[i-1]) {
                // 从当前的第二个开始，如果和前一个数重复则跳过
                continue;
            }
            tmp.push_back(candidates[i]);
            dfs(candidates, i+1, target-candidates[i]); // 进行回溯
            tmp.pop_back();
        }        
    }
};
```

Java代码

```Java
class Solution {
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    List<Integer> tmp = new ArrayList<Integer>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        // 先进行排序
        Arrays.sort(candidates);
        dfs(candidates, 0, target);
        return res;
    }

    public void dfs(int[] candidates, int cur, int target) {
        if (target == 0) {
            // 选取路径上和为目标，加入答案，结束递归
            res.add(new ArrayList<Integer>(tmp));
            return;
        }

        for (int i = cur; i < candidates.length && target-candidates[i] >= 0; ++i) {
            if (i > cur && candidates[i] == candidates[i-1]) {
                // 从当前的第二个开始，如果和前一个数重复则跳过
                continue;
            }
            tmp.add(candidates[i]);
            // 进行回溯
            dfs(candidates, i+1, target-candidates[i]);
            tmp.remove(tmp.size()-1);
        }
    }
}
```

### 083 没有重复元素集合的全排列

方法：额外使用一个和数组同样大的bool数组，标记一个元素是否被访问过，进入回溯时，将其标记为访问过，退出回溯时，再将其置为没有访问过

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;
    vector<bool> used;
    vector<vector<int>> permute(vector<int>& nums) {
        used.resize(nums.size(), false);
        dfs(nums, 0);
        return res;
    }

    void dfs(vector<int>& nums, int deep) {
        if (deep == nums.size()) {
            // 遍历深度和数组一样大，穷举完成，加入答案
            res.push_back(tmp);
        }

        for (int i = 0; i < nums.size(); ++i) {
            if (used[i]) {
                continue;
            }
            used[i] = true; // 访问过该数字
            tmp.push_back(nums[i]);
            dfs(nums, deep+1);
            tmp.pop_back();
            used[i] = false;
        }
    }
};
```

Java代码

```Java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> tmp = new ArrayList<>();
    boolean[] used;
    public List<List<Integer>> permute(int[] nums) {
        used = new boolean[nums.length];
        dfs(nums, 0);
        return res;
    }

    public void dfs(int[] nums, int deep) {
        if (deep == nums.length) {
            // 遍历深度和数组一样大，穷举完成，加入答案
            res.add(new ArrayList<>(tmp));
            return;
        }

        for (int i = 0; i < nums.length; ++i) {
            if (used[i]) {
                continue;
            }
            tmp.add(nums[i]);
            used[i] = true; // 访问过该数字
            dfs(nums, deep+1);
            used[i] = false;
            tmp.remove(tmp.size()-1);
        }
    }
}
```

### 084 含有重复元素集合的全排列

方法：类似082和083题， 先将集合排序，使用一个大小等于集合大小的数组记录集合中的数是否访问过。当回溯中当前的数字访问过，或者与当前数字相等的前一个数字没有被访问过，说明组合会重复，则跳过这个本次遍历（在重复数组中，只有一个被访问过而另一个没被访问过，才会被记录为一个组合）

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;
    vector<bool> visited;
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        visited.resize(nums.size(), false);
        // 先将数组排序，这样重复元素会连续
        sort(nums.begin(), nums.end());
        dfs(nums, 0);
        return res;
    }

    void dfs(vector<int>& nums, int deep) {
        if (deep == nums.size()) {
            // 路径深度和集合大小一样，完成一个组合，加入答案结束递归
            res.push_back(tmp);
            return ;
        }
        for (int i = 0; i < nums.size(); ++i) {
            if (visited[i] || (i > 0 && nums[i] == nums[i-1] && !visited[i-1])) {
                // 当回溯中当前的数字访问过，或者与当前数字相等的前一个数字没有被访问过，
                // 说明组合会重复，则跳过这个本次遍历
                continue;
            }
            visited[i] = true;  // 标记为访问过
            tmp.push_back(nums[i]);
            dfs(nums, deep+1);
            tmp.pop_back();
            visited[i] = false;
        }
    }
};
```

Java代码

```Java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> tmp = new ArrayList<>();
    boolean[] visited;
    public List<List<Integer>> permuteUnique(int[] nums) {
        // 先将数组排序，这样重复元素会连续
        Arrays.sort(nums);
        visited = new boolean[nums.length];
        dfs(nums, 0);
        return res;
    }

    public void dfs(int[] nums, int deep) {
        if (deep == nums.length) {
            // 路径深度和集合大小一样，完成一个组合，加入答案结束递归
            res.add(new ArrayList<>(tmp));
            return ;
        }

        for (int i = 0; i < nums.length; ++i) {
            if (visited[i] || (i > 0 && nums[i] == nums[i-1] && !visited[i-1])) {
                // 当回溯中当前的数字访问过，或者与当前数字相等的前一个数字没有被访问过，
                // 说明组合会重复，则跳过这个本次遍历
                continue;
            }
            visited[i] = true; // 标记为访问过
            tmp.add(nums[i]);
            dfs(nums, deep+1);
            tmp.remove(tmp.size()-1);
            visited[i] = false;
        }
    }
}
```

### 085 生成匹配括号

方法：采用类似二叉树的结构，递归加入左括号或右括号，并用两个变量记录左括号数量和右括号数量，当左右括号符合匹配要求时，加入答案退出此层递归

C++代码：

```C++
class Solution {
public:
    vector<string> res;
    string tmp;
    int left = 0;
    int right = 0;
    vector<string> generateParenthesis(int n) {
        // n对括号，对应2n个括号
        dfs(2*n, 0);
        return res;
    }

    void dfs(int n, int deep) {
        if (left == right && deep == n) {
            // 左右括号匹配且恰好为n对
            res.push_back(tmp);
            return;
        }
        if (left < right || left > n/2) {
            // 当左括号数量小于右括号，或者左括号长度大于 n/2说明不可能生成有效匹配
            return;
        }
        // 加入左括号
        tmp += "(";
        ++left;
        dfs(n, deep+1);
        --left;
        tmp.pop_back();
        // 加入右括号
        tmp += ")";
        ++right;
        dfs(n, deep+1);
        --right;
        tmp.pop_back();
    }
};
```

Java代码

```Java
class Solution {
    private List<String> res = new ArrayList<>();
    private StringBuilder tmp = new StringBuilder();
    int left = 0;
    int right = 0;
    public List<String> generateParenthesis(int n) {
        // n对括号，对应2n个括号
        dfs(2*n, 0);
        return res;
    }

    private void dfs(int n, int deep) {
        if (left == right && deep == n) {
            // 左右括号匹配且恰好为n对
            res.add(tmp.toString());
            return;
        }
        if (left < right || left > n / 2) {
            // 当左括号数量小于右括号，或者左括号长度大于 n/2说明不可能生成有效匹配
            return ;
        }
        // 加入左括号
        tmp.append('(');
        ++left;
        dfs(n, deep+1);
        --left;
        tmp.deleteCharAt(tmp.length()-1);
        // 加入右括号
        tmp.append(')');
        ++right;
        dfs(n, deep+1);
        --right;
        tmp.deleteCharAt(tmp.length()-1);
    }
}
```

### 086 分割回文子字符串

方法：穷举字符串的每种分割方式，并对不能构成回文子串的路径进行剪枝。

C++代码：

```C++
class Solution {
public:
    vector<vector<string>> res;
    vector<string> tmp;
    vector<vector<string>> partition(string s) {
        dfs(s, 0);
        return res;
    }

    void dfs(string& s, int idx) {
        if (idx == s.size()) {
            // 遍历到结尾，加入答案，退出当前层递归
            res.push_back(tmp);
            return;
        }
        // 穷举每个子字符串的分割
        for (int i = idx; i < s.size(); ++i) {
            string subs = s.substr(idx, i + 1 - idx);
            if (!ishui(subs)) {
                // 此子字符串不是回文，所以不能构成所有子串都是回文串的集合
                continue;
            }
            tmp.push_back(subs);
            dfs(s, i+1);
            tmp.pop_back();
        }
    }

    // 判断一个字符串是否是回文串
    bool ishui(string& s) {
        int left = 0;
        int right = s.size()-1;
        while (left <= right) {
            if (s[left] != s[right]) {
                return false;
            }
            ++left;
            --right;
        }
        return true;
    }
};
```

Java代码

```Java
class Solution {
    List<List<String>> res = new ArrayList<>();
    List<String> tmp = new ArrayList<>();
    public String[][] partition(String s) {
        dfs(s, 0);
        int rows = res.size();
        String[][] ret = new String[rows][];
        // 将List转化为答案要求的数组形式
        for (int i = 0; i < rows; ++i) {
            int cols = res.get(i).size();
            ret[i] = new String[cols];
            for (int j = 0; j < cols; ++j) {
                ret[i][j] = res.get(i).get(j);
            }
        }
        return ret;
    }

    public void dfs(String s, int idx) {
        if (idx == s.length()) {
            // 遍历到结尾，加入答案，退出当前层递归
            res.add(new ArrayList<String>(tmp));
            return;
        }
        // 穷举每个子字符串的分割
        for (int i = idx; i < s.length(); ++i) {
            String subs = s.substring(idx, i+1);
            if (!ishui(subs)) {
                // 此子字符串不是回文，所以不能构成所有子串都是回文串的集合
                continue;
            }
            tmp.add(subs);
            dfs(s, i+1);
            tmp.remove(tmp.size()-1);
        }
    }

    // 判断一个字符串是否是回文串
    public boolean ishui(String s) {
        int left = 0;
        int right = s.length() - 1;
        while (left <= right) {
            if (s.charAt(left) != s.charAt(right)) {
                return false;
            }
            ++left;
            --right;
        }
        return true;
    }
}
```

### 087 复原IP

方法：类似086 将字符串穷举分割，再判断是否能构成有效ip从而进行剪枝

C++代码：

```C++
class Solution {
public:
    vector<string> res;
    vector<string> tmp;
    vector<string> restoreIpAddresses(string s) {
        // 字符串大小小于3，构不成四个整数，大于12，则必有一个数大于255
        if (s.size() <= 3 || s.size() > 12) {
            return res;
        }
        dfs(s, 1, 0);
        return res;
    }

    void dfs(string& s, int deep, int idx) {
        if (idx == s.size() && tmp.size() == 4) {
            // 如果遍历到了结尾，并且4个ip地址数字都符合，则拼接ip地址放入答案
            string ip;
            for (auto& t: tmp) {
                ip += t;
                ip += ".";
            }
            ip.pop_back();// 去掉最后加入的 .
            res.push_back(ip);
            return;
        }
        // 深度超过四，不符合ip地址，退出本次递归
        if (deep > 4) {
            return;
        }
        for (int i = idx; i < s.size() && i < idx + 3; ++i) {
            string subs = s.substr(idx, i - idx + 1);
            if (subs[0] == '0' && subs.size() > 1) {
                // 以0开头，不符合，退出本次递归
                break;
            }
            int num = 0;
            if (s[idx] != 0) {
                num = stoi(subs);
            }
            if (num >255 || num < 0) {
                // 数值错误，不符合，重新分割
                continue;
            }
            tmp.push_back(subs);
            dfs(s, deep+1, i+1);
            tmp.pop_back();
        }
    }
};
```

Java代码

```Java
class Solution {
    List<String> res = new ArrayList<>();
    List<String> tmp = new ArrayList<>();
    public List<String> restoreIpAddresses(String s) {
        // 字符串大小小于3，构不成四个整数，大于12，则必有一个数大于255
        if (s.length() <= 3 || s.length() > 12) {
            return res;
        }
        dfs(s, 1, 0);
        return res;
    }

    public void dfs(String s, int deep, int idx) {
        if (tmp.size() == 4 && idx == s.length()) {
            // 如果遍历到了结尾，并且4个ip地址数字都符合，则拼接ip地址放入答案
            res.add(String.join(".", tmp));
            return;
        }
        // 深度超过四，不符合ip地址，退出本次递归
        if (deep > 4) {
            return;
        }
        for (int i = idx; i < s.length() && i < idx + 3; ++i) {
            String subs = s.substring(idx, i+1);
            if (subs.charAt(0) == '0' && subs.length() > 1) {
                // 以0开头，不符合，退出本次递归
                break;
            }
            int num = Integer.parseInt(subs);
            if (num > 255) {
                // 数值错误，不符合，重新分割
                break;
            }
            tmp.add(subs);
            dfs(s, deep+1, i+1);
            tmp.remove(tmp.size()-1);
        }
    }
}
```

## 动态规划

动态规划问题的一般形式是求最值。求解动态规划的核心问题是穷举，但动态规划的穷举优点特别，这类问题存在重叠子问题，具有“最优子结构”，暴力穷举效率低，所以需要“备忘录”或者“DP table”来优化穷举过程

动态规划的一个思维框架：
明确base case ==》 明确“状态” ==》明确“选择” ==》定义dp数组/函数的含义

```python
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

字符串问题，常常从从下标0开始长度为1到n的子串开始进行动态规划

0-1背包问题: 有n个物品nums，对应不同价值vals，背包容量为capacity，最多装多少价值物品

```C++
// base case是dp[i][0]和dp[0][j],即没有物品或没有空间时，为0
// dp[i][j] 表示对于前n个物品，当容量为j时能装下的最大价值
for (i = 0; i < n; ++i) {
    for (j = 1; j <= capacity; ++j) {
        if (j - nums[i-1] < 0) {
            // 背包容量不够，选择不装入背包
            dp[i][j] = dp[i-1][w];
        } else {
            // 装入或者不装入背包，择优
            dp[i][j] = max(dp[i-1][j-nums[i-1]] + val[i-1],
                            dp[i-1][j]);
        }
    }
} 
```

子集背包问题：一个数组能够取几个数，和恰好为sum

```C++
// base case是dp[i][0]为true，相当于和为0，不取即可恰好为0
// dp[0][j]为false，没有数组不可能和为j
// dp[i][j] 表示对于前i个物品，当容量为j时能否恰好装满
for (i = 0; i < n; ++i) {
    for (j = 1; j <= sum; ++j) {
        if (j - nums[i-1] < 0) {
            // 背包容量不够，选择不装入背包
            dp[i][j] = dp[i-1][j];
        } else {
            // 装入或者不装入背包，看是否能装满
            dp[i][j] = dp[i-1][j] || dp[i-1][j-nums[i-1]];
        }
    }
} 
```

完全背包问题：不同数值的数组，每个数可以取无限次，取出其中任意个数的和为sum有几种取法

```C++
// base case是dp[0][j] = 0,即不用任何数，无法凑出sum，dp[i][0]=1，任何个数去0有一种凑法
// dp[i][j] 表示只使用前i个物品，和为j时有几种凑法
for (i = 0; i < n; ++i) {
    for (j = 1; j <= sum; ++j) {
        if (j - nums[i-1] < 0) {
            // 背包容量不够，选择不装入背包
            dp[i][j] = dp[i-1][j];
        } else {
            // 两种选择之和
            dp[i][j] = dp[i-1][j] + dp[i][j-nums[i-1]];
        }
    }
} 
```

动态规划往往能将dp数组转化为滚动数组等形式减少空间复杂度

### 088 爬楼梯的最少成本

方法：动态规划，每次能爬一步或两步，所以爬第i个楼梯最小成本为第i-1个和第i-2个楼梯最小成本加上本层消耗成本。

C++代码：

```C++
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int n = cost.size();
        vector<int> dp(n, 0);
        // base case，爬一个或两个都是直接到达
        dp[0] = cost[0];
        dp[1] = cost[1];
        for (int i = 2; i < n; ++i) {
            // 动态转移方程
            dp[i] = min(dp[i-1], dp[i-2]) + cost[i];
        }
        return min(dp[n-1], dp[n-2]);
    }
};
```

Java代码

```Java
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        int n = cost.length;
        // 滚动数组优化
        int pre = cost[0];
        int cur = cost[1];
        for (int i = 2; i < n; ++i) {
            int next = Math.min(cur, pre) + cost[i];
            pre = cur;
            cur = next;
        }
        return Math.min(cur, pre);
    }
}
```

### 089 房屋盗窃

方法：动态规划，对于每个房屋，选择偷前一个或不偷前一个中的最大值

C++代码：

```C++
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.size() == 1) {
            return nums[0];
        }
        if (nums.size() == 2) {
            return max(nums[0], nums[1]);
        }
        int n = nums.size();
        vector<int> dp(n);
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for (int i = 2; i < n; ++i) {
            // 选择偷前一个或不偷前一个
            dp[i] = max(dp[i-1], dp[i-2] + nums[i]);
        }
        return dp[n-1];
    }
};
```

Java代码

```Java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) {
            return nums[0];
        }
        // 滚动数组优化
        int pre = nums[0];
        int cur = Math.max(nums[0], nums[1]);
        for (int i = 2; i < n; ++i) {
            int tmp = cur;
            cur = Math.max(cur, pre + nums[i]);
            pre = tmp;
        }
        return cur;
    }
}
```

### 090 环形房屋偷窃

方法：类似上一题，分别偷第0个到n-2个，或者1到n-1个房屋，取其中最大值

C++代码：

```C++
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n == 1) {
            return nums[0];
        }
        // 将环拆开分层两个线性部分进行偷窃，这样不会有头尾相邻问题
        return max(rob(nums, 0, n-2), rob(nums, 1, n-1));
    }

    // 从指定位置开始偷到结束位置
    int rob(vector<int>& nums, int start, int end) {
        int pre = 0;
        int cur = 0;
        for (int i = start; i <= end; ++i) {
            int tmp = cur;
            cur = max(cur, pre + nums[i]);
            pre = tmp;
        }
        return cur;
    }
};
```

Java代码

```Java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) {
            return nums[0];
        }
        // 将环拆开分层两个线性部分进行偷窃，这样不会有头尾相邻问题
        return Math.max(rob(nums, 0, n-2), rob(nums, 1, n-1));
    }

    // 从指定位置开始偷到结束位置
    public int rob(int[] nums, int start, int end) {
        int pre = 0;
        int cur = 0;
        for (int i = start; i <= end; ++i) {
            int tmp = cur;
            cur = Math.max(cur, pre + nums[i]);
            pre = tmp;
        }
        return cur;
    }
}
```

### 091 粉刷房子

方法：由于当已知粉刷前 ii 个房子的最小花费成本时，根据粉刷第 i + 1i+1 号房子的花费成本可以计算粉刷前 i + 1i+1 个房子的最小花费成本，因此可以使用动态规划计算最小花费成本。由于每个房子可以被粉刷成三种颜色中的一种，因此需要分别考虑粉刷成三种颜色时的最小花费成本。

C++代码：

```C++
class Solution {
public:
    int minCost(vector<vector<int>>& costs) {
        int n = costs.size();
        if (costs.size() == 1) {
            // 只有一个房子，选择三种颜色最低的
            return min(costs[0][0], min(costs[0][1], costs[0][2]));
        }
        // 表示粉刷第0号房子到第i号房子且第i号房子被粉刷成第j种颜色时的最小花费成本
        vector<vector<int>> dp(n, vector<int>(3));
        dp[0] = costs[0];
        for (int i = 1; i < n; ++i) {
            // 每种颜色的动态转移方程
            dp[i][0] = min(dp[i-1][1], dp[i-1][2]) + costs[i][0];
            dp[i][1] = min(dp[i-1][0], dp[i-1][2]) + costs[i][1];
            dp[i][2] = min(dp[i-1][0], dp[i-1][1]) + costs[i][2];
        }
        // 选择粉刷到最后三种颜色中，成本最低的
        return min(dp[n-1][0], min(dp[n-1][1], dp[n-1][2]));
    }
};
```

Java代码

```Java
class Solution {
    public int minCost(int[][] costs) {
        int n = costs.length;
        if (n == 1) {
            // 只有一个房子，选择三种颜色最低的
            return Math.min(costs[0][0], Math.min(costs[0][1], costs[0][2]));
        }
        
        // 使用滚动数组进行优化
        int pre0 = costs[0][0];
        int pre1 = costs[0][1];
        int pre2 = costs[0][2];
        int cur0 = 0;
        int cur1 = 0;
        int cur2 = 0;
        for (int i = 1; i < n; ++i) {
            cur0 = Math.min(pre1, pre2) + costs[i][0];
            cur1=  Math.min(pre0, pre2) + costs[i][1];
            cur2 = Math.min(pre0, pre1) + costs[i][2];
            pre0 = cur0;
            pre1 = cur1;
            pre2 = cur2;
        }
        // 选择粉刷到最后三种颜色中，成本最低的
        return Math.min(cur0, Math.min(cur1, cur2));
    }
}
```

### 092 翻转字符

方法：由于字符串s的每个位置的字符可以是o或1，因此对于每个位置需要分别计算该位置的字符是0和该位置的字符是1的情况下的最小翻转次数

C++代码：

```C++
class Solution {
public:
    int minFlipsMonoIncr(string s) {
        int n = s.size();
        // dp[i][0]和dp[i][1]分别表示下标i处为1或0的情况下使得s递增的最小翻转次数
        vector<vector<int>> dp(n, vector<int>(2));
        dp[0][1] = 0;
        dp[0][0] = s[0] == '1'? 1: 0;

        for (int i = 1; i < n; ++i) {
            // 如果前面都有翻转为0，当前页翻转为0保持递增
            dp[i][0] = dp[i-1][0] + (s[i] == '1'? 1: 0);
            // 如果要翻转为1，选择前面递增的少翻转次数，即前面为0，不用翻转，或者为1，需要翻转
            dp[i][1] = min(dp[i-1][0], dp[i-1][1] + (s[i] == '0'? 1: 0));
        }
        // 返回全翻转为0或者翻转为1的最小值
        return min(dp[n-1][0], dp[n-1][1]);
    }
};
```

Java代码

```Java
class Solution {
    public int minFlipsMonoIncr(String s) {
        int n = s.length();
        // 滚动数组优化
        int dp0 = s.charAt(0) == '0'? 0: 1;
        int dp1 = 0;
        for (int i = 1; i < n; ++i) {
            // 这里要注意顺序，防止覆盖变量
            dp1 = Math.min(dp0, dp1 + (s.charAt(i) == '0'? 1: 0));
            dp0 = dp0 + (s.charAt(i) == '0'? 0: 1);
        }
        return Math.min(dp1, dp0);
    }
}
```

### 093 最长斐波那契数列

方法：状态定义dp\[i][j]为以i，j为结尾的斐波那契数列最大长度，于是如果存在数字arr[k],使得arr[i] - arr[j] = arr[k]，有dp\[i][j] = dp\[j][k] + 1，为了快速找到这个k，可以先用一个HashMap记下数字的下标。注意最小的长度为3

C++代码：

```C++
class Solution {
public:
    int lenLongestFibSubseq(vector<int>& arr) {
        int n = arr.size();
        int res = 0;
        // dp[i][j]定义为以下标i，j结尾的斐波那契数列长度
        vector<vector<int>>dp(n, vector<int>(n));
        // 找k时加快速度避免遍历数组，先记下对应位置
        unordered_map<int, int> mp;
        for (int i = 0; i < n; ++i) {
            mp[arr[i]] = i;
        }
        // 两两遍历数组
        for (int i = 2; i < n; ++i) {
            for (int j = i - 1; j >= 0 && j + 2 > res; --j) {
                if (arr[i] - arr[j] >= arr[j]) {
                    // 这种情况不存在一个递增的斐波那契数列
                    break;
                }
                // 存在这样arr[i] - arr[j] = arr[k]
                if (mp.count(arr[i] - arr[j]) != 0) {
                    int k = mp[arr[i] - arr[j]];
                    dp[i][j] = max(3, dp[j][k] + 1); // 最短序列长度为3
                    // 记下遍历过程中的最大值
                    res = max(res, dp[i][j]);
                }
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int lenLongestFibSubseq(int[] arr) {
        int n = arr.length;
        int res = 0;
        // 找k时加快速度避免遍历数组，先记下对应位置
        Map<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < n; ++i) {
            map.put(arr[i], i);
        }
        // dp[i][j]定义为以下标i，j结尾的斐波那契数列长度
        int[][] dp = new int[n][n];
        // 两两遍历数组
        for (int i = 2; i < n; ++i) {
            for (int j = i - 1; j >= 0 && j + 2 > res; --j) {
                if (arr[i] - arr[j] >= arr[j]) {
                    // 这种情况不存在一个递增的斐波那契数列
                    break;
                }
                // 存在这样arr[i] - arr[j] = arr[k]
                if (map.containsKey(arr[i] - arr[j])) {
                    int k = map.get(arr[i] - arr[j]);
                    dp[i][j] = Math.max(3, dp[j][k] + 1);// 最短序列长度为3
                    // 记下遍历过程中的最大值
                    res = Math.max(res, dp[i][j]);
                }
            }
        }
        return res;
    }
}
```

### 094 最少回文分割

方法：进行两次动态规划，首先用动态规划穷举出每个子串是否是是回文，在进行动态规划，寻找最小分割次数

C++代码：

```C++
class Solution {
public:
    int minCut(string s) {
        int n = s.size();
        // dp[i][j] 表示i到j这一段字符串是否为回文串
        // 默认只有一个字符，为回文串
        vector<vector<bool>> dp(n, vector<bool>(n, true));
        for (int i = n-2; i >= 0; --i) {
            for (int j = i + 1; j < n; ++j) {
                dp[i][j] = s[i] == s[j] && dp[i+1][j-1];
            }
        }
        // f[i]代表将0到i这一段分割成若干回文子串所需的最小分割次数
        // 字符串最长为2000，先设定默认最大分割为2000
        vector<int> f(n, 2000);
        for (int i = 0; i < n; ++i) {
            // 如果0到i满足回文，则不需要分割，分割数为0
            if (dp[0][i]) {
                f[i] = 0;
            } else {
                // 遍历长度为0到i之间可以分割的方案中的最小值
                for (int j = 0; j < i; ++j) {
                    if (dp[j + 1][i]) {
                        f[i] = min(f[i], f[j] + 1);
                    }
                }
            }
        }
        return f[n-1];
    }
};
```

Java代码

```Java
class Solution {
    // 看答案的动态规划
    public int minCut(String s) {
        int n = s.length();
        // dp[i][j] 表示i到j这一段字符串是否为回文串
        // 默认只有一个字符，为回文串
        boolean[][] dp = new boolean[n][n];
        for (int i = 0; i < n; ++i) {
            Arrays.fill(dp[i], true);
        }
        for (int i = n - 2; i >= 0; --i) {
            for (int j = i + 1; j < n; ++j) {
                dp[i][j] = s.charAt(i) == s.charAt(j) && dp[i+1][j-1];
            }
        }

        // f[i]代表将0到i这一段分割成若干回文子串所需的最小分割次数
        int[] f = new int[n];
        for (int i = 0; i < n; ++i) {
            if (dp[0][i]) {
                // 如果0到i满足回文，则不需要分割，分割数为0
                f[i] = 0;
            } else {
                f[i] = 2000;// 字符串最长为2000，先设定默认最大分割为2000
                // 遍历长度为0到i之间可以分割的方案中的最小值
                for (int j = 0; j < i; ++j) {
                    if (dp[j+1][i]) {
                        f[i] = Math.min(f[i], f[j] + 1);
                    }
                }
            }
        }
        return f[n-1];
    }
}
```

### 095 最长公共子序列

方法：采用动态规划，两个个字符串分别从下标0到1至0到n个的子串开始计算最长公共子序列

C++代码：

```C++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size();
        int n = text2.size();
        // dp[i][j] 表示第一个字符串前i个字符和第二个字符串前j个字符最长公共子序列，默认为0
        // 为了方便计算，多开一个dp长度，循环从1开始
        vector<vector<int>> dp(m+1, vector<int>(n+1));// [0:i-1]  [0:j-1]
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (text1[i-1] == text2[j-1]) {
                    // 两个字符串最后一个字符相对，公共子序列长度加1
                    dp[i][j] = dp[i-1][j-1]+1;
                } else {
                    // 最后一个字符不相等，取两者分别去掉最后一个字符后的最长子序列
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
                }
            }
        }
        return dp[m][n];
    }
};
```

Java代码

```Java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        // dp[i][j] 表示第一个字符串前i个字符和第二个字符串前j个字符最长公共子序列，默认为0
        // 为了方便计算，多开一个dp长度，循环从1开始
        int[][] dp = new int[m+1][n+1];
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (text1.charAt(i-1) == text2.charAt(j-1)) {
                    // 两个字符串最后一个字符相对，公共子序列长度加1
                    dp[i][j] = dp[i-1][j-1] + 1;
                } else {
                    // 最后一个字符不相等，取两者分别去掉最后一个字符后的最长子序列
                    dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
                }
            }
        }
        return dp[m][n];
    }
}
```

### 096 字符串交织

方法：使用动态规划，可以将两个字符串作为行和列画出矩阵，看是否存在一条路径，能组成s3
注意本题不能使用双指针，因为两字符重复部分不法区分移动哪一个指针

C++代码：

```C++
class Solution {
public:
    bool isInterleave(string s1, string s2, string s3) {
        int m = s1.size();
        int n = s2.size();
        int t = s3.size();
        if (m + n != t) {
            // 长度不满足要求，直接返回false
            return false;
        }
        // f[i][j]表示第一个字符串前i个字符和第二个字符串前j个字符能否组成第三个字符串的前i+j个字符
        vector<vector<bool>> f(m+1, vector<bool>(n+1, false));
        // base case为0，即0个字符可以组成0个字符的字符串
        f[0][0] = true;
        for (int i = 0; i <= m; ++i) {
            for (int j = 0; j <= n; ++j) {
                if (i > 0) {
                    // s1的前i-1个可以组成，且s1的第i个和s3的第i+j-1个相等，则可以组成
                    f[i][j] = f[i][j] || (f[i-1][j] && s1[i-1] == s3[i + j - 1]);
                }
                if (j > 0) {
                    // s2的前j-1个可以组成，且s2的第j个和s3的第i+j-1个相等，则可以组成
                    f[i][j] = f[i][j] || (f[i][j-1] && s2[j-1] == s3[i + j - 1]);
                }
            }
        }
        return f[m][n];  
    }
};
```

Java代码

```Java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int m = s1.length();
        int n = s2.length();
        int t = s3.length();
        if (m + n != t) {
            // 长度不满足要求，直接返回false
            return false;
        } 
        // f[i][j]表示第一个字符串前i个字符和第二个字符串前j个字符能否组成第三个字符串的前i+j个字符
        boolean[][] f = new boolean[m+1][n+1];
        // base case为0，即0个字符可以组成0个字符的字符串
        f[0][0] = true;
        for (int i = 0; i <=m; ++i) {
            for (int j = 0; j <= n; ++j) {
                if (i > 0) {
                    // s1的前i-1个可以组成，且s1的第i个和s3的第i+j-1个相等，则可以组成
                    f[i][j] = f[i][j] || (f[i-1][j] && s1.charAt(i-1) == s3.charAt(i+j-1));
                }
                if (j > 0) {
                    // s2的前j-1个可以组成，且s2的第j个和s3的第i+j-1个相等，则可以组成
                    f[i][j] = f[i][j] || (f[i][j-1] && s2.charAt(j-1) == s3.charAt(i+j-1));
                }
            }
        }
        return f[m][n];
    }
}
```

### 097 子序列的数目

方法：将题目看成是从s中挑出子序列能组成t，比较最后一个字符是否相等，进行动态规划

C++代码：

```C++
class Solution {
public:
    int numDistinct(string s, string t) {
        int m = s.size();
        int n = t.size();
        if (m < n) {
            return 0;
        }
        // dp[i][j]表示s前i个字符可以组成t前j个字符的个数
        // 防止中间结果数值溢出，需要unsigned long long
        vector<vector<unsigned long long>> dp(m+1, vector<unsigned long long>(n+1, 0)); 
        // base case dp[i][0] = 1,dp[0][j] = 0;
        for (int i = 0; i <= m; ++i) {
            dp[i][0] = 1;
        }
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= i && j <= n; ++j) {
                if (s[i-1] == t[j-1]) {
                    // dp[i][j]等于算上最后一个字符的个数，加上不算上最后一个字符的个数
                    dp[i][j] = dp[i-1][j-1] + dp[i-1][j];
                } else {
                    // 最后一个字符不相等，继承不算上最后一个字符的个数
                    dp[i][j] = dp[i-1][j];
                }
            }
        }
        return dp[m][n];
    }
};
```

Java代码

```Java
class Solution {
    public int numDistinct(String s, String t) {
        int m = s.length();
        int n = t.length();
        if (m < n) {
            return 0;
        }
        // dp[i][j]表示s前i个字符可以组成t前j个字符的个数
        int[][] dp = new int[m+1][n+1];
        for (int i = 0; i <= m; ++i) {
            dp[i][0] = 1;
        }
        // base case dp[i][0] = 1,dp[0][j] = 0;
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= i && j <= n; ++j) {
                if (s.charAt(i-1) == t.charAt(j-1)) {
                    // dp[i][j]等于算上最后一个字符的个数，加上不算上最后一个字符的个数
                    dp[i][j] = dp[i-1][j-1] + dp[i-1][j];
                } else {
                    // 最后一个字符不相等，继承不算上最后一个字符的个数
                    dp[i][j] = dp[i-1][j];
                }
            }
        }
        return dp[m][n];
    }
}
```

### 098 路径的数目

方法：走到每个位置的路径数目，等于从上走到这个位置和从左走到这个位置的路径数之和

C++代码：

```C++
class Solution {
public:
    int uniquePaths(int m, int n) {
        // dp[i][j]表示走到坐标(i,j)的路径数目
        // base case为dp[i][0] = 1,dp[0][j] = 1,即只往右或只往下走只有一种
        vector<vector<int>> dp(m, vector<int>(n, 1));
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                // 到达一个点的路径数，等于从上走下和从左走右的路径数之和
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
};
```

Java代码

```Java
class Solution {
    public int uniquePaths(int m, int n) {
        // dp[i][j]表示走到坐标(i,j)的路径数目
        int[][] dp = new int[m][n];
        // base case为dp[i][0] = 1,dp[0][j] = 1,即只往右或只往下走只有一种
        for (int i = 0; i < m; ++i) {
            dp[i][0] = 1;
        }
        for (int i = 0; i < n; ++i) {
            dp[0][i] = 1;
        }
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                // 到达一个点的路径数，等于从上走下和从左走右的路径数之和
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
}
```

### 099 最小路径之和

方法：类似上一题，走到每个位置的最小路径之和，等于从上走下来和从左走过来的路径中最小的路径和。

C++代码：

```C++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(); 
        int n = grid[0].size();
        // dp[i][j]表示走到坐标(i,j)的最小路径
        vector<vector<int>> dp(m, vector<int>(n));
        int sum = 0;
        // 从开始位置只往右走或只往下走路径只有一天
        // base case 为最左和最上的那一排的路径和
        for (int i = 0; i < m; ++i) {
            sum += grid[i][0];
            dp[i][0] = sum;
        }
        sum = 0;
        for (int i = 0; i < n; ++i) {
            sum += grid[0][i];
            dp[0][i] = sum;
        }
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                // 走到每个位置的最小路径之和，等于从上走下来和从左走过来的路径中最小的路径和。
                dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[m-1][n-1];
    }
};
```

Java代码

```Java
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        // dp[i][j]表示走到坐标(i,j)的最小路径
        int[][] dp = new int[m][n];
        int sum = 0;
        // 从开始位置只往右走或只往下走路径只有一天
        // base case 为最左和最上的那一排的路径和
        for (int i = 0; i < m; ++i) {
            sum += grid[i][0];
            dp[i][0] = sum;
        }
        sum = 0;
        for (int i = 0; i < n; ++i) {
            sum += grid[0][i];
            dp[0][i] = sum;
        }
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                // 走到每个位置的最小路径之和，等于从上走下来和从左走过来的路径中最小的路径和。
                dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[m-1][n-1];
    }
}
```

### 100 三角形中最小路径之和

方法：每个点的最小路径，等于该点上一层左右两边的最小路径之和加上该点的路径值，需要注意最左边和最右边的节点

C++代码：

```C++
class Solution {
public:
    int minimumTotal(vector<vector<int>>& triangle) {
        int n = triangle.size();
        // dp[i]表示到第i层的各个节点的最小路径，最大层节点个数为n，预先分配空间
        // 这里已近将其转化为从上层到下层的二维压缩成一维的动态规划
        vector<int> dp(n);
        // base case为最开始位置的一个节点路径值
        dp[0] = triangle[0][0];
        for (int i = 1; i < n; ++i) {
            for (int j = i; j >= 0 ; --j) {
                if (j == i) {
                    // 三角形最右边只有一条路径
                    dp[j] = dp[j-1] + triangle[i][j];
                } else if (j == 0) {
                    // 三角形最左边只有一条路径
                    dp[j] += triangle[i][j];
                } else {
                    // 取上一层左右两边的最小路径
                    dp[j] = min(dp[j], dp[j-1]) + triangle[i][j];
                }    
            }
        }
        int res = INT_MAX;
        // 找出最后一层最小路径
        for (int a: dp) {
            res = min(a, res);
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        int n = triangle.size();
        // dp[i]表示到第i层的各个节点的最小路径，最大层节点个数为n，预先分配空间
        // 这里已近将其转化为从上层到下层的二维压缩成一维的动态规划
        int[] dp = new int[n];
        // base case为最开始位置的一个节点路径值
        dp[0] = triangle.get(0).get(0);
        for (int i = 1; i < n; ++i) {
            // 三角形最右边只有一条路径
            dp[i] = dp[i-1] + triangle.get(i).get(i);
            for (int j = i-1; j > 0; --j) {
                // 取上一层左右两边的最小路径
                dp[j] = Math.min(dp[j], dp[j-1]) + triangle.get(i).get(j);
            }
            // 三角形最左边只有一条路径
            dp[0] += triangle.get(i).get(0);
        }     
        // 找出最后一层最小路径 
        int res = Integer.MAX_VALUE;
        for (int a: dp) {
            res = Math.min(res, a);
        } 
        return res;
    }
}
```

### 101 分割等和子集

方法：将等和分割装换为背包容量为总和一般的子集背包问题

C++代码：

```C++
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();
        int sum = 0;
        for (int num: nums) {
            sum += num;
        }
        // 元素和为奇数，不可能分割
        if ((sum & 1) == 1) {
            return false;
        }
        // 装换成子集背包问题，取出的数值恰好等于数组总和的一半
        int target = sum / 2;
        // dp[i][j]表示前i个数字能否找出凑成j的子集
        vector<vector<bool>> dp(n, vector<bool>(target+1, false));
        // base case 第0个数一个就可以组成和为nums[0]，即dp[0][nums[0]] = true
        // dp[i][0] = false, 即正整数和都不会为0
        if (nums[0] <= target) {
            dp[0][nums[0]] = true;
        }
        for (int i = 1; i < n; ++i) {
            for(int j = 0; j <= target; ++j) {
                if (nums[i] > j) {
                    // 当前值大于j，看之前的数字能否组成j
                    dp[i][j] = dp[i-1][j];
                }
                if (nums[i] == j) {
                    // 当前一个数就可以组成
                    dp[i][j] = true;
                }
                if (nums[i] < j) {
                    // 不取或取时，能否组成
                    dp[i][j] = dp[i-1][j] || dp[i-1][j-nums[i]];
                }
            }
        }
        return dp[n-1][target];
    }
};
```

Java代码

```Java
class Solution {
    public boolean canPartition(int[] nums) {
        int n = nums.length;
        int sum = 0;
        for(int num: nums) {
            sum += num;
        }
        if ((sum & 1) == 1) {
            return false;
        }
        int target = sum / 2;
        boolean[] dp = new boolean[target+1];
        // 滚动数组优化
        dp[0] = true;
        if (nums[0] <= target) {
            dp[nums[0]] = true;
        }
        for (int i = 1; i < n; ++i) {
            // 注意遍历位置需要从后往前，否则会出现覆盖问题
            for (int j = target; j >= 0 && j >= nums[i]; --j) {
                dp[j] = dp[j] || dp[j-nums[i]];
            }
            if (dp[target]) {
                return true;
            }
        }
        return dp[target];
    }
}
```

### 102 加减的目标值

方法：假设数组元素和为sum，负数和为neg，则有(sum-neg)-neg = sum - 2*neg = target，于是将问题转化成，从数组中选几个值添加负号，使得这些值的和为neg的子集背包问题

C++代码：

```C++
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for (auto num: nums) {
            sum += num;
        }
        if (sum < target) {
            // 全部为正都小于target，直接返回0
            return 0;
        }
        if ((sum - target) & 1 == 1) {
            return 0;
        }
        // 转化为子集背包问题后的目标和
        int neg = (sum - target) / 2;
        int n = nums.size(); 
        // dp[i][j]表示，数组前i个数的子序列和恰好为j的组合个数
        vector<vector<int>> dp(n+1, vector<int>(neg+1, 0));
        // base case dp[0][0] = 1,dp[0][j] = 0
        // 当没有任何元素可以选取时，元素和只能是0，对应的方案数是1
        dp[0][0] = 1;
        for (int i = 1; i <= n; ++i) {
            for (int j = 0; j <= neg; ++j) {
                if (nums[i-1] > j) {
                    dp[i][j] = dp[i-1][j];
                } else if (nums[i-1] <= j) {
                    // 不取的当前值个数加上取当前值的个数
                    dp[i][j] = dp[i-1][j] + dp[i-1][j-nums[i-1]];
                }
            }
        }
        return dp[n][neg];
    }
};
```

Java代码

```Java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int sum = 0;
        for (int num: nums) {
            sum += num;
        }
        if (sum < target || (sum - target) % 2 == 1) {
            // 全部为正都小于target，直接返回0
            return 0;
        }
        // 转化为子集背包问题后的目标和
        int neg = (sum - target) / 2;
        // 滚动形式优化空间
        int[] dp = new int[neg+1];
        dp[0] = 1;
        for (int i = 1; i <= nums.length; ++i) {
            // 注意遍历从后开始，避免覆盖
            for (int j = neg; j >= nums[i-1]; --j) {
                dp[j] += dp[j-nums[i-1]];
            }
        }
        return dp[neg];
    }
}
```

### 103 最少的硬币数目

方法：完全背包问题

C++代码：

```C++
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        // dp[i]表示组成金额为i需要的最少硬币
        // 这是完全背包问题压缩成一维，默认为无穷多，这里去数据范围上限10001
        vector<int> dp(amount+1, 10001);
        dp[0] = 0;
        for (int i = 1; i <= amount; ++i) {
            for (int coin: coins) {
                if (coin <= i) {
                    // 不取或取当前的硬币，选择最少的数量
                    dp[i] = min(dp[i], dp[i-coin]+1);
                }
            }
        }
        // 不存在组合，返回-1
        return dp[amount] == 10001? -1: dp[amount];
    }
};
```

Java代码

```Java
class Solution {
    public int coinChange(int[] coins, int amount) {
        // dp[i]表示组成金额为i需要的最少硬币
        // 这是完全背包问题压缩成一维，默认为无穷多，这里去数据范围上限10001
        int[] dp = new int[amount+1];
        Arrays.fill(dp, 10001);
        dp[0] = 0;
        for (int i = 1; i <= amount; ++i) {
            for (int coin: coins) {
                if (coin <= i) {
                    // 不取或取当前的硬币，选择最少的数量
                    dp[i] = Math.min(dp[i], dp[i-coin]+1);
                }
            }
        }
        // 不存在组合，返回-1
        return dp[amount]== 10001? -1: dp[amount];
    }
}
```

### 104 排列的数目

方法：完全背包问题

C++代码：

```C++
class Solution {
public:
    int combinationSum4(vector<int>& nums, int target) {
        // 完全背包问题，dp[i]表示和为i的组合个数，中间结果或超过int，所以用long long
        // 已经使用滚动形式优化了空间复杂度
        vector<unsigned long long> dp(target+1);
        dp[0] = 1;
        for (int i = 1; i <= target; ++i) {
            for (int num: nums) {
                if (num <= i) {
                    // 不取的数目加上取的数目
                    dp[i] += dp[i-num];
                }
            }
        }
        return dp[target];
    }
};
```

Java代码

```Java
class Solution {
    public int combinationSum4(int[] nums, int target) {
        // 完全背包问题，dp[i]表示和为i的组合个数
        // 已经使用滚动形式优化了空间复杂度
        int[] dp = new int[target+1];
        dp[0] = 1;
        for (int i = 1; i <= target; ++i) {
            for (int num: nums) {
                if (num <= i) {
                    // 不取的数目加上取的数目
                    dp[i] += dp[i-num];
                }
            }
        }
        return dp[target];
    }
}
```

## 图

图的题的常用方法有：深度、广度优先搜索，并查集，拓扑排序

并查集（不相交集合，主要用于一些分组的问题，管理一系列不相交的集合，用于处理动态连通性问题，并查集在查询过程中动态改变。
并查集只回答两个节点是不是在一个连通分量中（也就是所谓的连通性问题），并不回答路径问题
如果一个问题具有传递性质，可以考虑用并查集。并查集最常见的一种设计思想是：把同一个连通分量中的节点组织成一个树形结构

一般来说，会将每个元素设定一个从0开始的id，在使用一个parent[n]的数组(或者哈希表)存放每个下标为id的元素指向的父母的id，如果有权重，还会有一个weight[n]标记每个下标为id的元素的权重。

初始时parent[i] = i;

查的操作有两种：路径压缩(隔代压缩)和按秩合并

```C++
// 路径压缩(隔代压缩)
int find(int x) {
    if (x != parent[x]) {
        parent[x] = find(parent[x]);
    }
    return parent[x];
}
// 按秩合并
int find(int x) {
    while (x != parent[x]) {
        parent[x] = parent[parent[x]];
        x = parent[s];
    }
    return x;
}
```

合并操作，把一个集合的根节点指向另一个集合的根结点，只要根结点一样，就表示在一个集合里

```C++
void union(int x, int y) {
    int rootX = find(x);
    int rootY = find(y);
    if (rootX == rootY) {
        return;
    }
    parent[rootX] = rootY;
    // 或一行
    parent[find(x)] = find(y);
}
```

拓扑排序是广度优先遍历和贪心搜索应用于有向图的一个专有名词
拓扑排序的作用：得到一个“拓扑序”，拓扑序不唯一。 检测有向图是否有环（无向图检测是否有环，可用并查集）
拓扑排序算法流程

```C++
vector<int> findOrder(int num, vector<vector<int>>& prerequisites) {
    // 结点映射为数组下标，记录每个结点的入度
    vector<int> inDegree(num, 0);   
    // 记下每个结点的邻接节点，即记下本节点指向了哪些节点
    unordered_map<int, vector<int>> adj;
    for (auto& v: prerequisites) {
        adj[v[1]].push_back(v[0]);
        ++inDegree[v[0]];
    }
    // 在开始排序前，扫描对应的存储空间（使用邻接表），将入度为0的结点放入队列。
    queue<int> que;
    for (int i = 0; i < num; ++i) {
        if (inDegree[i] == 0) {
            que.push(i);
        }
    }
    vector<int> res;
    // 只要队列非空，就从队首取出入度为 00 的结点，将这个结点输出到结果集中
    while (!que.empty()) {
        int node = que.front();
        que.pop();
        res.push_back(node);
        // 将这个结点的所有邻接结点（它指向的结点）的入度减1
        for (int inode: adj[node]) {
            --inDegree[inode];
            // 在减1以后，如果这个被减1的结点的入度为0，就继续入队。
            if (inDegree[inode] == 0) {
                que.push(inode);
            }
        }
    }
    // 当队列为空的时候，检查结果集中的顶点个数是否和节点个数相等
    if (res.size() == num) {
        return res;
    }
    return {};
}
```

### 105 岛屿的最大面积

方法：使用一个和图一样大的数组，用来记录图中每个坐标是否被访问过，对每个坐标进行深度优先遍历，遇到岛屿地区则继续递归，遇到海水地区则结束递归。这样在每次访问到岛屿的一格，就可以访问完整个岛屿

C++代码：

```C++
class Solution {
public:
    int res = 0;
    int tmp = 0;
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();
        // 标记是否访问过的数组
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        // 遍历每个格子
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 1 && !visited[i][j]) {
                    tmp = 0; // 一开始面积记为0
                    dfs(grid, visited, tmp, i, j);
                    // 取最大岛屿
                    res = max(res, tmp);
                }
            }
        }
        return res;
    }

    // 注意这里area是引用传递
    void dfs(vector<vector<int>>& grid, vector<vector<bool>>& visited, int& area, int i , int j) {
        if (i < 0 || i >= grid.size() || j < 0 || j >= grid[0].size() || visited[i][j]) {
            return;
        }
        ++area;
        visited[i][j] = true; // 记下访问过
        // 向上遍历
        if (i-1 >=0 && grid[i-1][j] == 1) {
            dfs(grid, visited, area, i-1, j);
        }
        // 向下遍历
        if (i+1 < grid.size() && grid[i+1][j] == 1) {
            dfs(grid, visited, area, i+1, j);
        }
        // 向左遍历
        if (j-1 >=0 && grid[i][j-1] == 1) {
            dfs(grid, visited, area, i, j-1);
        }
        // 向右遍历
        if (j+1 < grid[0].size() && grid[i][j+1] == 1) {
            dfs(grid, visited, area, i, j+1);
        }
    }
};
```

Java代码

```Java
class Solution {
    public int area = 0;
    public int maxAreaOfIsland(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int res = 0;
        // 标记是否访问过的数组
        boolean[][] visited = new boolean[m][n];
        // 遍历每个格子
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (!visited[i][j] && grid[i][j] == 1) {
                    area = 0;// 一开始面积记为0
                    dfs(grid, visited, i, j);
                    res = Math.max(res, area);// 取最大岛屿
                }
            }
        }
        return res;
    }

    public void dfs(int[][] grid, boolean[][] visited, int i, int j) {
        int m = grid.length;
        int n = grid[0].length;
        if (i < 0 || i >=m || j < 0 || j >= n || visited[i][j]) {
            return;
        }
        visited[i][j] = true;// 记下访问过
        ++area;
        // 向上遍历
        if (i-1 >= 0 && grid[i-1][j] == 1) {
            dfs(grid, visited, i-1, j);
        }
        // 向下遍历
        if (i+1 < m && grid[i+1][j] == 1) {
            dfs(grid, visited, i+1, j);
        }
        // 向左遍历
        if (j-1 >= 0 && grid[i][j-1] == 1) {
            dfs(grid, visited, i, j-1);
        }
        // 向右遍历
        if (j+1 < n && grid[i][j+1] == 1) {
            dfs(grid, visited, i, j+1);
        }
    }
}
```

### 106 二分图

方法：由于二分图每一条边的两个结点属于两个集合，所以可以采用上色的方式，将相邻的两个节点标记为不同颜色。从某个节点开始，采用深度优先遍历将邻接的节点标记为不同颜色，如果遍历到某个节点已经上过色，且与当前邻接节点颜色相同（与要标记的颜色不同），说明不是二分图

C++代码：

```C++
class Solution {
public:
    bool isBipartite(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> visited(n);
        // 对每个结点进行遍历
        for (int i = 0; i < n; ++i) {
            if (visited[i] == 0) {
                if (dfs(graph, visited, 1, i) == false) {
                    // 不是二分图，直接返回false
                    return false;
                }
            }
        }
        return true;
    }

    bool dfs(vector<vector<int>>& graph, vector<int>& visited, int c, int idx) {
        if (visited[idx] != 0) {
        // 要标记的颜色和已经标记的颜色是否相同
        return visited[idx] == c;
        }
        visited[idx] = c;
        // 深度优先遍历邻接的所有节点
        for (int i: graph[idx]) {
            if (!dfs(graph, visited, -c, i)) {
                // 不是二分图，直接返回false
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
    public boolean isBipartite(int[][] graph) {
        int n = graph.length;
        int[] visited = new int[n];
        // 对每个结点进行遍历
        for (int i = 0; i < n; ++i) {
            if (visited[i] == 0) {
                if (!dfs(graph, visited, 1, i)) {
                    // 不是二分图，直接返回false
                    return false;
                }
            }
        }
        return true;
    }

    boolean dfs(int[][] graph, int[] visited, int c, int idx) {
        if (visited[idx] != 0) {
            // 要标记的颜色和已经标记的颜色是否相同
            return visited[idx] == c;
        }
        visited[idx] = c;
        // 深度优先遍历邻接的所有节点
        for (int i: graph[idx]) {
            if (!dfs(graph, visited, -c, i)) {
                // 不是二分图，直接返回false
                return false;
            }
        }
        return true;
    }
}
```

### 107 矩阵中的距离

方法：先将mat为0的位置加入广度优先遍历的队列，并将对应答案置为0；从每个为0的的格子向外使用广度优先遍历未被访问过的格子，遍历时，周围每一圈的距离是内圈的距离加1

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
        int m = mat.size();
        int n = mat[0].size();
        // 预先设置距离为-1，标记为没访问过
        vector<vector<int>> res(m, vector<int>(n, -1));
        queue<pair<int, int>> q;
        // 遍历数组，先将mat为0的位置加入广度优先遍历的队列，并将对应答案置为0
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (mat[i][j] == 0) {
                    res[i][j] = 0;
                    q.push(make_pair(i, j));
                }
            }
        }
        // 遍历的四个方向
        int dx[4] = {-1, 1, 0, 0};
        int dy[4] = {0, 0, -1, 1};
        while (!q.empty()) {
            auto p = q.front();
            q.pop();
            int x = p.first;
            int y = p.second;
            // 向四个方向向外遍历
            for (int i = 0; i < 4; ++i) {
                int x2 = x + dx[i];
                int y2 = y + dy[i];
                if (x2 >= 0 && x2 < m && y2 >=0 && y2 < n && res[x2][y2] == -1) {
                    // 如果没访问过，则为上一个为1的格子到现在的距离+1
                    res[x2][y2] = res[x][y] + 1;
                    q.push(make_pair(x2, y2)); // 加入遍历队列
                }
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int[][] updateMatrix(int[][] mat) {
        Queue<int[]> queue = new LinkedList<>();
        int m = mat.length;
        int n = mat[0].length;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (mat[i][j] == 0) {
                    // 遍历数组，先将mat为0的位置加入广度优先遍历的队列
                    queue.offer(new int[]{i, j});
                } else {
                    // 预先设置距离为-1，标记为没访问过
                    mat[i][j] = -1;
                }
            }
        }
        // 遍历的四个方向
        int[] dx = new int[]{-1,1,0,0};
        int[] dy = new int[]{0,0,-1,1};
        while(!queue.isEmpty()) {
            int[] point = queue.poll();
            int x = point[0];
            int y = point[1];
            // 向四个方向向外遍历
            for (int i = 0; i < 4; ++i) {
                int x2 = x + dx[i];
                int y2 = y + dy[i];
                if (x2 >= 0 && x2 < m && y2 >= 0 && y2 < n && mat[x2][y2] == -1) {
                    // 如果没访问过，则为上一个为1的格子到现在的距离+1
                    mat[x2][y2] = mat[x][y] + 1;
                    queue.offer(new int[]{x2, y2}); // 加入遍历队列
                }
            }
        }
        return mat;
    }
}
```

### 108 单词演变

方法：从beginWord开始，对每个字母进行25个字母（除自己）的替换，看替换是否还在单词列表中，若还在，则这是一条路径。采用广度优先遍历，并按层次遍历，找到到达endWord的路径（因为是按层次，最先到达的就是最短路径）

C++代码：

```C++
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        int n = beginWord.size();
        // 先记下字符串列表，减少查找时的时间复杂度
        unordered_set<string> st;
        for (auto& w: wordList) {
            st.insert(w);
        }
        // 集合中没有endWord,就不可能转换到endWord
        if (st.count(endWord) == 0) {
            return 0;
        }
        unordered_set<string> visited;// 记下遍历过的字符串
        visited.insert(beginWord);  // 从beginWord开始
        queue<string> que;
        que.push(beginWord);
        int step = 0;
        // 广度优先遍历
        while (!que.empty()) {
            // 类似层次遍历，遍历上一层相连（可转换）的下一层
            int queSize = que.size();
            ++step;
            for (int i = 0; i < queSize; ++i) {
                string word = que.front();
                que.pop();
                // 对当前单词每一个字母进行替换，看能够进行装换
                for (int j = 0; j < n; ++j) {
                    char c = word[j];
                    for (int k = 'a'; k <= 'z'; ++k) {
                        if (k == c) {
                            continue;
                        }
                        word[j] = k; // 替换一个字母
                        if (word == endWord) {
                            // 到达endWord，返回步数
                            return step + 1;
                        }
                        // 如果可以替换，且未被访问过，则加入遍历队列，标记为访问过
                        if (st.count(word) != 0 && visited.count(word) == 0) {
                            que.push(word);
                            visited.insert(word);
                        }
                    }
                    // 替换回原来的字母
                    word[j] = c;
                }
            }
        }
        return 0;
    }
};
```

Java代码

```Java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        // 先记下字符串列表，减少查找时的时间复杂度
        Set<String> set = new HashSet<>(wordList);
        // 集合中没有endWord,就不可能转换到endWord
        if (!set.contains(endWord)) {
            return 0;
        }
        Queue<String> queue = new LinkedList<>();
        queue.offer(beginWord);
        Set<String> visited = new HashSet<>();// 记下遍历过的字符串
        // 从beginWord开始
        visited.add(beginWord);
        int step = 0;
        // 广度优先遍历
        while (!queue.isEmpty()) {
            // 类似层次遍历，遍历上一层相连（可转换）的下一层
            int queSize = queue.size();
            ++step;
            for (int i = 0; i < queSize; ++i) {
                String word = queue.poll();
                // 对当前单词每一个字母进行替换，看能够进行装换
                for (int j = 0; j < beginWord.length(); ++j) {
                    char[] cword = word.toCharArray();
                    char c = cword[j];
                    for (int k = 'a'; k <= 'z'; ++k) {
                        if (c == k) {
                            continue;
                        }
                        cword[j] = (char)k; // 替换一个字母
                        String nextWord = new String(cword);
                        if (set.contains(nextWord) && !visited.contains(nextWord)) {
                            // 如果可以替换，且未被访问过，则加入遍历队列，标记为访问过
                            if (endWord.equals(nextWord)) {
                                // 到达endWord，返回步数
                                return step + 1;
                            }
                            visited.add(nextWord);
                            queue.offer(nextWord);
                        }
                    }
                }
            }
        }
        return 0;
    }
}
```

### 109 开密码锁

方法：类似上一题，采用广度优先遍历，遍历每次拨动密码可以产生的路径。

C++代码：

```C++
class Solution {
public:
    unordered_set<string> mp;
    int openLock(vector<string>& deadends, string target) {
        if (target == "0000") {
            return 0;
        }
        // 用哈希表记下deadends，加快查找速度
        for (string& s: deadends) {
            mp.insert(s);
        }
        // deadends中有0000，一开始就不可能
        if (mp.count("0000")) {
            return -1;
        }

        unordered_set<string> visited;  // 访问过的数字组合
        // 从0000开始
        visited.insert("0000");
        // 存放的是当前数字，当前步数
        queue<pair<string,int>> que; 
        que.push(make_pair("0000", 0));
        // 广度优先搜索
        while (!que.empty()) {
            auto [state, step] = que.front();
            que.pop();
            // 对一次拨动转换的密码进行遍历
            for (string& next_state: get(state)) {
                // 如果没访问过，且不再deadends中
                if (visited.count(next_state)==0 && mp.count(next_state) == 0) {
                    if (next_state == target) {
                        // 搜索到了目标答案，返回步数
                        return step+1;
                    }
                    // 将下一步遍历加入队列
                    que.push(make_pair(next_state, step+1));
                    visited.insert(next_state);
                }
            }
        }
        return -1;
    }

    // 返回一个密码组合可以拨动一次变成的各种的密码组合列表
    vector<string> get(string& state) {
        vector<string> ret;
        for (int i = 0; i < 4; ++i) {
            char num = state[i];
            state[i] = num + 1 > '9'? '0': num+1;
            ret.push_back(state);
            state[i] = num - 1 < '0'? '9': num-1;
            ret.push_back(state);
            state[i] = num;
        }
        return ret;
    }
};
```

Java代码

```Java
class Solution {
    public int openLock(String[] deadends, String target) {
        // 一开始就是答案
        if ("0000".equals(target)) {
             return 0;
        }
        // 用哈希表记下deadends，加快查找速度
        Set<String> dead = new HashSet<>();
        for (String s: deadends) {
            dead.add(s);
        }
        // deadends中有0000，一开始就不可能
        if (dead.contains("0000")) {
            return -1;
        }
        int step = 0;
        // 从0000开始
        Queue<String> queue = new LinkedList<>();
        queue.offer("0000");
        Set<String> visited = new HashSet<>();
        visited.add("0000");
        // 广度优先搜索
        while (!queue.isEmpty()) {
            // 类似于层次遍历，这样方便记下步数
            ++step;
            int size = queue.size();
            for (int i = 0; i < size; ++i) {
                String state = queue.poll();
                // 对一次拨动转换的密码进行遍历
                for (String nextState: get(state)) {
                    if (!visited.contains(nextState) && !dead.contains(nextState)) {
                        if (target.equals(nextState)) {
                            // 搜索到了目标答案，返回步数
                            return step;
                        }
                        // 将下一步遍历加入队列
                        queue.offer(nextState);
                        visited.add(nextState);
                    }
                }
            }
        }
        return -1;
    }
    // 返回一个密码组合可以拨动一次变成的各种的密码组合列表
    public List<String> get(String state) {
        List<String> ret = new ArrayList<>();
        char[] cstate = state.toCharArray();
        for (int i = 0; i < 4; ++i) {
            char num = cstate[i];
            cstate[i] = num + 1 > '9'? '0': (char)(num+1);
            ret.add(new String(cstate));
            cstate[i] = num - 1 < '0'? '9': (char)(num-1);
            ret.add(new String(cstate));
            cstate[i] = num;
        }
        return ret;
    }
}
```

### 110 所有路径

方法：采用回溯法，遍历所有路径

C++代码：

```C++
class Solution {
public:
    vector<vector<int>> res;
    vector<int> tmp;
    vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
        // 从0开始
        tmp.push_back(0);
        dfs(graph, 0);
        return res;
    }

    void dfs(vector<vector<int>>& graph, int idx) {
        if (idx == graph.size()-1) {
            // 到达结尾，加入答案并返回
            res.push_back(tmp);
            return;
        }
        // 对邻接节点进行遍历
        for (int a: graph[idx]) {
            tmp.push_back(a);
            dfs(graph, a);
            tmp.pop_back();
        }
    }
};
```

Java代码

```Java
class Solution {
    List<List<Integer>> res = new ArrayList<>();
    List<Integer> tmp = new ArrayList<>();
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        // 从0开始
        tmp.add(0);
        dfs(graph, 0);
        return res;
    }

    public void dfs(int[][] graph, int idx) {
        if (idx == graph.length - 1) {
            // 到达结尾，加入答案并返回
            res.add(new ArrayList<Integer>(tmp));
            return;
        }
        // 对邻接节点进行遍历
        for (int a: graph[idx]) {
            tmp.add(a);
            dfs(graph, a);
            tmp.remove(tmp.size()-1);
        }
    } 
}
```

### 111 计算除法

方法：使用并查即，对除数和被除数当作节点，两数的倍数当作权重。

C++代码：

```C++
class Solution {
public:
    vector<int> parent; //
    vector<double> weight;
    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        int eSize = equations.size();   // 等式的个数
        parent.resize(2 * eSize);// 每个等式两个数，所以最多有2倍个数的数字
        weight.resize(2 * eSize);
        // 初始化指向自己
        for (int i = 0; i < 2 * eSize; ++i) {
            parent[i] = i;
            weight[i] = 1.0;
        }
        // 为每个等式的除数和被除数设定id
        unordered_map<string, int> mp;
        int id = 0;
        for (int i = 0; i < eSize; ++i) {
            vector<string> v = equations[i];
            if (mp.find(v[0]) == mp.end()) {
                mp[v[0]] = id;
                ++id;
            }
            if (mp.find(v[1]) == mp.end()) {
                mp[v[1]] = id;
                ++id;
            }
            // 对被除数和除数做并操作
            unions(mp[v[0]], mp[v[1]], values[i]);
        }
        // 进行查询
        int qsize = queries.size();
        vector<double> res(qsize, -1.0);
        for (int i = 0; i < qsize; ++i) {
            vector<string> v = queries[i];
            if (mp.find(v[0]) != mp.end() && mp.find(v[1]) != mp.end()) {
                res[i] = isConnect(mp[v[0]], mp[v[1]]);
            }
        }
        return res;
    }
    
    // 并操作
    void unions(int x, int y, double value) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) {
            return;
        }
        parent[rootX] = rootY;
        // 改变权重
        weight[rootX] = weight[y] * value / weight[x]; 
    }

    // 查操作
    int find(int x) {
        if (x != parent[x]) {
            int orign = parent[x];
            parent[x] = find(parent[x]);
            weight[x] *= weight[orign];
        }
        return parent[x];
    }

    // 判断是否是连接
    double isConnect(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) {
            // 在同一个集合内，说明存在倍数关系
            return weight[x] / weight[y];
        }
        return -1.0;
    }
};
```

Java代码

```Java
class Solution {
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
        int equationsSize = equations.size();

        UnionFind unionFind = new UnionFind(2 * equationsSize);
        // 1、预处理，将变量的值与id进行映射，是的并查集底层使用数组实现，方便编码
        Map<String, Integer> hashMap = new HashMap<>(2 * equationsSize);
        int id = 0;
        for (int i = 0; i < equationsSize; ++i) {
            String var1 = equations.get(i).get(0);
            String var2 = equations.get(i).get(1);
            if (!hashMap.containsKey(var1)) {
                hashMap.put(var1, id);
                ++id;
            }
            if (!hashMap.containsKey(var2)) {
                hashMap.put(var2, id);
                ++id;
            }
            // 对被除数和除数做并操作
            unionFind.union(hashMap.get(var1), hashMap.get(var2), values[i]);
        }
        // 2、做查询
        int queriesSize = queries.size();
        double[] res = new double[queriesSize];
        for (int i = 0; i < queriesSize; ++i) {
            String var1 = queries.get(i).get(0);
            String var2 = queries.get(i).get(1);
            Integer id1 = hashMap.get(var1);
            Integer id2 = hashMap.get(var2);
            if (id1 == null || id2 == null) {
                res[i] = -1.0;
            } else {
                res[i] = unionFind.isConnected(id1, id2);
            }
        }
        return res;
    }
    // 并查集的类
    private class UnionFind {
        private int[] parent;
        private double[] weight;
        public UnionFind(int n) {
            this.parent = new int[n];
            this.weight = new double[n];
            for (int i = 0; i < n; ++i) {
                parent[i] = i;
                weight[i] = 1.0d;
            }
        }
        public void union(int x, int y, double value) {
            int rootX = find(x);
            int rootY = find(y);
            if (rootX == rootY) {
                return;
            }
            parent[rootX] = rootY;
            weight[rootX] = weight[y] * value / weight[x];
        }

        public int find(int x) {
            if (x != parent[x]) {
                int origin = parent[x];
                parent[x] = find(parent[x]);
                weight[x] *= weight[origin];
            }
            return parent[x];
        }

        public double isConnected(int x, int y) {
            int rootX = find(x);
            int rootY = find(y);
            if (rootX == rootY) {
                return weight[x] / weight[y];
            } else {
                return -1.0d;
            }
        }
    }
}
```

### 112 最长递增路径

方法：将矩阵看成一个有向图，每个单元格对应图中的一个节点，如果相邻的两个单元格的值不相等，则在相邻的两个单元格之间存在一条从较小值指向较大值的有向边。问题转化成在有向图中寻找最长路径。使用记忆化深度优先搜索，当访问到一个单元格(i,j)时，如果memo\[i][j] = memo\[i][j]，说明该单元格的结果已经计算过，则直接从缓存中读取结果，如果memo\[i][j]=0，说明该单元格的结果尚未被计算过，则进行搜索，并将计算得到的结果存入缓存中。

C++代码：

```C++
class Solution {
public:
    int dx[4] = {-1, 1, 0, 0};
    int dy[4] = {0, 0, -1, 1};
    int longestIncreasingPath(vector<vector<int>>& matrix) {
        int m = matrix.size();
        int n = matrix[0].size();
        vector<vector<int>> memo(m, vector<int>(n, 0));
        int res = 0;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                // 遍历每个位置开始的路径，取最大值
                res = max(res, dfs(matrix, memo, i, j));
            }
        }
        return res;
    }

    int dfs(vector<vector<int>>& matrix, vector<vector<int>>& memo, int x, int y) {
        if (memo[x][y] != 0) {
            // 已经遍历过，返回缓存的结果
            return memo[x][y];
        }
        int m = matrix.size();
        int n = matrix[0].size();
        ++memo[x][y];   // 没有遍历过，路径为1
        for (int i = 0; i < 4; ++i) {
            int x1 = x + dx[i];
            int y1 = y + dy[i];
            if (x1 >= 0 && x1 < m && y1 >= 0 && y1 < n && matrix[x1][y1] > matrix[x][y]) {
                // 往递增方向遍历，当前的最长路径为四个方向的最长递增路径加1，即此格子
                memo[x][y] = max(memo[x][y], dfs(matrix, memo, x1, y1) + 1);
            }
        }
        return memo[x][y];
    }
};
```

Java代码

```Java
class Solution {
    public int[] dx = new int[]{-1, 1, 0, 0};
    public int[] dy = new int[]{0, 0, -1, 1};
    public int longestIncreasingPath(int[][] matrix) {
        int m = matrix.length;
        int n = matrix[0].length;
        int[][] memo = new int[m][n];
        int res = 0;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                // 遍历每个位置开始的路径，取最大值
                res = Math.max(res, dfs(matrix, memo, i, j));
            }
        }
        return res;

    }

    public int dfs(int[][] matrix, int[][] memo, int x, int y) {
        if (memo[x][y] != 0) {
            // 已经遍历过，返回缓存的结果
            return memo[x][y];
        }
        int m = matrix.length;
        int n = matrix[0].length;
        ++memo[x][y]; // 没有遍历过，路径为1
        for (int i = 0; i < 4; ++i) {
            int x1 = x + dx[i];
            int y1 = y + dy[i];
            if (x1 >= 0 && x1 < m && y1 >= 0 && y1 < n && matrix[x1][y1] > matrix[x][y]) {
                // 往递增方向遍历，当前的最长路径为四个方向的最长递增路径加1，即此格子
                memo[x][y] = Math.max(memo[x][y], dfs(matrix, memo, x1, y1) + 1);
            }
        }
        return memo[x][y];
    }
}
```

### 113 课程顺序

方法：采用拓扑排序，看排序完之后是否能完成所有课程

C++代码：

```C++
class Solution {
public:
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        // 结点映射为数组下标，记录每个结点的入度
        vector<int> inDegree(numCourses, 0);
        // 记下每个结点的邻接节点，即记下本节点指向了哪些节点
        unordered_map<int, vector<int>> adj;
        for (auto& v: prerequisites) {
            adj[v[1]].push_back(v[0]);
            ++inDegree[v[0]];
        }
        // 在开始排序前，扫描对应的存储空间（使用邻接表），将入度为0的结点放入队列。
        queue<int> que;
        for (int i = 0; i < numCourses; ++i) {
            if (inDegree[i] == 0) {
                que.push(i);
            }
        }
        vector<int> res;
        // 只要队列非空，就从队首取出入度为 00 的结点，将这个结点输出到结果集中
        while (!que.empty()) {
            int node = que.front();
            que.pop();
            res.push_back(node);
            // 将这个结点的所有邻接结点（它指向的结点）的入度减1
            for (int inode: adj[node]) {
                --inDegree[inode];
                // 在减1以后，如果这个被减1的结点的入度为0，就继续入队。
                if (inDegree[inode] == 0) {
                    que.push(inode);
                }
            }
        }
        // 当队列为空的时候，检查结果集中的顶点个数是否和节点个数相等
        if (res.size() == numCourses) {
            return res;
        }
        return {};
    }
};
```

Java代码

```Java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        // 记下每个结点的邻接节点，即记下本节点指向了哪些节点
        Map<Integer, List<Integer>> adj = new HashMap<>();
        // 结点映射为数组下标，记录每个结点的入度
        int[] inDegree = new int[numCourses];
        for (int[] v: prerequisites) {
            if (!adj.containsKey(v[1])) {
                adj.put(v[1], new ArrayList<Integer>());
            }
            adj.get(v[1]).add(v[0]);

            ++inDegree[v[0]];
        }
        // 在开始排序前，扫描对应的存储空间（使用邻接表），将入度为0的结点放入队列。
        Queue<Integer> que = new LinkedList<>();
        for (int i = 0; i < numCourses; ++i) {
            if (inDegree[i] == 0) {
                que.offer(i);
            }
        }
        // 只要队列非空，就从队首取出入度为 00 的结点，将这个结点输出到结果集中
        int[] res = new int[numCourses];
        int count = 0;
        while (!que.isEmpty()) {
            int node = que.poll();
            res[count] = node;
            ++count;
            // 将这个结点的所有邻接结点（它指向的结点）的入度减1
            if (adj.containsKey(node)) {
                for (int inode: adj.get(node)) {
                    --inDegree[inode];
                    // 在减1以后，如果这个被减1的结点的入度为0，就继续入队。
                    if (inDegree[inode] == 0) {
                        que.offer(inode);
                    }
                }
            }  
        }
        // 当队列为空的时候，检查结果集中的顶点个数是否和节点个数相等
        if (count == numCourses) {
            return res;
        }
        return new int[0];
    }
}
```

### 114 外星文字典

方法：根据单词排列的顺序建立字母间的有向图结构，然后使用拓扑排序确定字母顺序

C++代码：

```C++
class Solution {
public:
    unordered_map<char, int> indegree;
    unordered_map<char, vector<char>> mp;   // first-->second
    bool valid = true;
    string alienOrder(vector<string>& words) {
        // 建立邻接关系
        for (auto& word: words) {
            for (int j = 0; j < word.size(); ++j) {
                char c = word[j];
                if (mp.count(c) == 0) {
                    mp[c] = vector<char>();
                }
            }
        }
        // 单词间建立边的关系
        for (int i = 1; i < words.size(); ++i) {
            addEdge(words[i-1], words[i]);
        }
        string res;
        if (!valid) {
            // 字典顺序冲突，直接返回
            return res;
        }
        // 入度为0的节点加入队列
        queue<int> que;
        for (auto [u, _]: mp) {
            if (indegree.count(u) == 0) {
                que.push(u);
                res.push_back(u);
            }
        }
        // 拓扑排序
        while (!que.empty()) {
            int qsize = que.size();
            for (int i = 0; i < qsize; ++i) {
                int node = que.front();
                que.pop();
                for (int inode: mp[node]) {
                    --indegree[inode];
                    if (indegree[inode] == 0) {
                        res.push_back(inode);
                        que.push(inode);
                    }
                }
            }
        }
        // 是否完成拓扑排序
        return res.size() == mp.size()? res: "";
    }

    // 根据两个单词，建立字典序的字母有向图
    void addEdge(string& before, string& after) {
        int m = before.size();
        int n = after.size();
        int i = 0;
        while (i < m && i < n) {
            // 直到第一个不相等的字母
            if (before[i] != after[i]) {
                mp[before[i]].push_back(after[i]);
                ++indegree[after[i]];
                break;
            }
            ++i;
        }
        // 如果不满足字典顺序，则标记后，在上面直接返回
        if ((i == m || i == n) && m > n) {
            valid = false;
        }
    }
};
```

Java代码

```Java
class Solution {
    public Map<Character, List<Character>> edges = new HashMap<>();
    public Map<Character, Integer> indegrees = new HashMap<>();
    boolean valid = true;
    public String alienOrder(String[] words) {
        // 记录邻接关系
        for (String word: words) {
            for (int j = 0; j < word.length(); ++j) {
                char c = word.charAt(j);
                edges.putIfAbsent(c, new ArrayList<Character>());
            }
        }
        // 单词间建立边的关系
        for (int i = 1; i < words.length && valid; ++i) {
            addEdge(words[i-1], words[i]);
        }
        if (!valid) {
            // 字典顺序冲突，直接返回
            return "";
        }
        // 入度为0的节点加入队列
        Queue<Character> queue = new LinkedList<>();
        Set<Character> letterSet = edges.keySet();
        for (char u: letterSet) {
            if (!indegrees.containsKey(u)) {
                queue.offer(u);
            }
        }
        StringBuffer order = new StringBuffer();
        while (!queue.isEmpty()) {
            char u = queue.poll();
            order.append(u);
            for (char v: edges.get(u)) {
                indegrees.put(v, indegrees.get(v) - 1);
                if (indegrees.get(v) == 0) {
                    queue.offer(v);
                }
            }
        }
        // 是否完成拓扑排序
        return order.length() == edges.size()? order.toString(): "";

    }

    // 根据两个单词，建立字典序的字母有向图
    public void addEdge(String before, String after) {
        int m = before.length();
        int n = after.length();
        int length = Math.min(m, n);
        int i = 0;
        while (i < length) {
            // 直到第一个不相等的字母
            char c1 = before.charAt(i);
            char c2 = after.charAt(i);
            if (c1 != c2) {
                edges.get(c1).add(c2);
                indegrees.put(c2, indegrees.getOrDefault(c2, 0) + 1);
                break;
            }
            ++i;
        }
        // 如果不满足字典顺序，则标记后，在上面直接返回
        if (i == length && m > n) {
            valid = false;
        }

    }
}
```

### 115 重建序列

方法：将序列构建成有向图，唯一最短超序列，具有唯一的拓扑排序序列，也就是在拓扑排序的过程中，每次入度为0的节点只有一个

C++代码：

```C++
class Solution {
public:
    bool sequenceReconstruction(vector<int>& nums, vector<vector<int>>& sequences) {
        int n = nums.size();
        vector<int> indegree(n+1, 0);
        unordered_map<int, vector<int>> graph(n+1);
        for (int i = 1; i < n+1; ++i) {
            graph[i] = vector<int>();
        }
        for (auto& v: sequences) {
            for (int i = 1; i < v.size(); ++i) {
                graph[v[i-1]].push_back(v[i]);
                indegree[v[i]]++;
            }
        }
        queue<int> que;
        for (int i = 1; i <= n; ++i) {
            if (indegree[i] == 0) {
                que.push(i);
            }
        }
        while (!que.empty()) {
            // 唯一拓扑序，在每次排序入度为0的节点只有一个
            if (que.size() > 1) {
                return false;
            }
            int num = que.front();
            que.pop();
            for (int next: graph[num]) {
                --indegree[next];
                if (indegree[next] == 0) {
                    que.push(next);
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
    public boolean sequenceReconstruction(int[] nums, int[][] sequences) {
        int n = nums.length;
        int[] indegree = new int[n+1];
        Map<Integer, List<Integer>>  map = new HashMap<>();
        for (int i = 1; i < n+1; ++i) {
            map.put(i, new ArrayList<>());
        }
        for (int[] seq: sequences) {
            for (int i = 1; i < seq.length; ++i) {
                map.get(seq[i-1]).add(seq[i]);
                ++indegree[seq[i]];
            }
        }
        Queue<Integer> queue = new LinkedList<>();
        for (int i = 1; i < n+1; ++i) {
            if (indegree[i] == 0) {
                queue.offer(i);
            }
        }
        while (!queue.isEmpty()) {
            // 唯一拓扑序，在每次排序入度为0的节点只有一个
            if (queue.size() > 1) {
                return false;
            }
            int num = queue.poll();
            for (int next: map.get(num)) {
                --indegree[next];
                if (indegree[next] == 0) {
                    queue.offer(next);
                }
            }
        }
        return true;
    }
}
```

### 116 省份数量

方法：使用并查集，对每个两连接的城市进行并操作，最后并查集中，自己指向自己的个数，即集合的个数，就表示省份的数量

C++代码：

```C++
class Solution {
public:
    vector<int> parent;
    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        parent.resize(n);
        // 初始情况下默认指向自己
        for (int i = 0; i < n; ++i) {
            parent[i] = i;
        }
        // 对连接的两个城市进行并操作
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (isConnected[i][j] == 1) {
                    uni(i, j);
                }
            }
        }
        int res = 0;
        // 自己指向自己的个数，即集合的个数，就表示省份的数量
        for (int i = 0; i < n; ++i) {
            if (parent[i] == i) {
                ++res;
            }
        }
        return res;
    }

    int find(int x) {
        if (x != parent[x]) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    void uni(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) {
            return;
        }
        parent[rootX] = rootY;
    }
};
```

Java代码

```Java
class Solution {

    public int[] parent;
    public int findCircleNum(int[][] isConnected) {
        int n =isConnected.length;
        parent = new int[n];
        // 初始情况下默认指向自己
        for (int i = 0; i < n; ++i) {
            parent[i] = i;
        }
        // 对连接的两个城市进行并操作
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (isConnected[i][j] == 1) {
                    uni(i, j);
                }
            }
        }
        int res = 0;
        // 自己指向自己的个数，即集合的个数，就表示省份的数量
        for (int i = 0; i < n; ++i) {
            if (parent[i] == i) {
                ++res;
            }
        }
        return res;

    }

    public int find(int x) {
        while (x != parent[x]) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    public void uni(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        parent[rootX] = rootY;
    }
}
```

### 117 相似的字符串

方法：使用并查集，将相似字符放到一个集合之中，然后判断有多少个集合

C++代码：

```C++
class Solution {
public:
    vector<int> parent;
    unordered_map<string, int> mp;
    int numSimilarGroups(vector<string>& strs) {
        int n = strs.size();
        parent.resize(n);
        for (int i = 0; i < n; ++i) {
            parent[i] = i;
        }
        int id = 0;
        // 为每个字符串分配id
        for (string& str: strs) {
            if (mp.find(str) == mp.end()) {
                mp[str] = id;
                ++id;
            }
        }
        // 两两比较，对相似字符进行并操作
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                if (isSim(strs[i], strs[j])) {
                    uni(mp[strs[i]], mp[strs[j]]);
                }
            }
        }
        int res = 0;
        // 判断有几个集合
        for (auto& [_, i]: mp){
            if (parent[i] == i) {
                ++res;
            }
        }

        return res;
    }

    // 判断两个字符串是不是相似
    bool isSim(string& s1, string& s2) {
        int cnt = 0;    // 不同字符个数
        for (int i = 0; i < s1.size(); ++i) {
            if (s1[i] != s2[i]) {
                ++cnt;
            }
            if (cnt > 2) {
                return false;
            }
        }
        return true;
    }

    int find(int x) {
        while (x != parent[x]) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return parent[x];
    }

    void uni(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) {
            return;
        }
        parent[rootX] = rootY;
    }
};
```

Java代码

```Java
class Solution {
    public int[] parent;
    Map<String, Integer> mp = new HashMap<>();
    public int numSimilarGroups(String[] strs) {
        int n = strs.length;
        parent = new int[n];
        int id = 0;
        // 为每个字符串分配id
        for (String str: strs) {
            if (!mp.containsKey(str)) {
                mp.put(str, id);
                ++id;
            }
        }
        for (int i = 0; i < n; ++i) {
            parent[i] = i;
        }
        // 两两比较，对相似字符进行并操作
        for (int i = 0; i < n; ++i) {
            for (int j = i+1; j < n; ++j) {
                if (isSim(strs[i], strs[j])) {
                    uni(mp.get(strs[i]), mp.get(strs[j]));
                }
            }
        }
        int res = 0;
        // 判断有几个集合
        for (int i: mp.values()) {
            if (parent[i] == i) {
                ++res;
            }
        }
        return res;
    }

    // 判断两个字符串是不是相似
    public boolean isSim(String s1, String s2) {
        int cnt = 0;
        for (int i = 0; i < s1.length(); ++i) {
            if (s1.charAt(i) != s2.charAt(i)) {
                ++cnt;   
            }
            if (cnt > 2) {
                return false;
            }
        }
        return true;
    }

    public int find(int x) {
        while (parent[x] != x) {
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return parent[x];
    }

    public void uni(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        parent[rootX] = rootY;
    }
}
```

### 118 多余的边

方法：使用并查集，如果两个点已经位于同一个并查集中，这条边就是多余的边

C++代码：

```C++
class Solution {
public:
    vector<int> parent;
    vector<int> res;
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();
        parent.resize(n+1);
        for (int i = 0; i < n+1; ++i) {
            parent[i] = i;
        }
        // 对每条边的两个节点进行并操作
        for (auto& v: edges) {
            uni(v[0], v[1]);
        }
        return res;
    }

    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    void uni(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) {
            // 当两个点已经位于同一个并查集中，这条边就是多余的边
            res = {x, y};
        }
        parent[rootX] = rootY;
    }
};
```

Java代码

```Java
class Solution {
    int[] parent;
    int[] res = new int[2];
    public int[] findRedundantConnection(int[][] edges) {
        int n = edges.length;
        parent = new int[n+1];
        for (int i = 0; i < n; ++i) {
            parent[i] = i;
        }
        // 对每条边的两个节点进行并操作
        for (int[] v: edges) {
            uni(v[0], v[1]);
        }
        return res;
    }

    private int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    private void uni(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);
        if (rootX == rootY) {
            // 当两个点已经位于同一个并查集中，这条边就是多余的边
            res[0] = x;
            res[1] = y;
        }
        parent[rootX] = rootY;
    }
}
```

### 119 最长连续序列

方法：对于每个连续子序列的第一个数开始，从头开始查看它的下一个值是否在列表中，在的话就继续查找下一个值。序列的第一个数减一的值一定不再set中，每次只从序列第一个数开始遍历可以避免重复

C++代码：

```C++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        // set一方面是去重，另一个方面是加速查找
        unordered_set<int> st;
        for (int num: nums) {
            st.insert(num);
        }
        int res = 0;
        for (int num: nums) {
            // 数组里没有num-1的数，表示是连续序列的第一个数，只从第一个数开始遍历
            if (st.count(num-1) == 0) {
                int len = 1;
                // 遍历序列的下一个数
                while (st.count(num+1) == 1) {
                    ++num;
                    ++len;
                }
                // 记下最长的连续序列
                res = max(len, res);
            }
        }
        return res;
    }
};
```

Java代码

```Java
class Solution {
    public int longestConsecutive(int[] nums) {
        // set一方面是去重，另一个方面是加速查找
        Set<Integer> st = new HashSet<>();
        for (int num: nums) {
            st.add(num);
        }
        int res = 0;
        for (int num: nums) {
            // 数组里没有num-1的数，表示是连续序列的第一个数，只从第一个数开始遍历
            if (!st.contains(num-1)) {
                int len = 1;
                // 遍历序列的下一个数
                while (st.contains(num+1)) {
                    ++num;
                    ++len;
                }
                // 记下最长的连续序列
                res = Math.max(res, len);
            }
        }
        return res;
    }
}
```

完结！
