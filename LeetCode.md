# LeetCode题解

[TOC]

## No.1 [寻找和为n的两个数](https://leetcode.com/problems/two-sum)

### 题目描述

给定一个数组 nums 和一个数 target，要求找到数组中和为 target 的两个数的坐标(测试案例均有解且**每个元素只能使用一次**)

### 解法

- 暴力搜索

  用两层循环暴力搜索解，时间复杂度为 O(n^2)。

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> res;
        int s = nums.size();
        for(int i = 0;i < s - 1;i++){
            for(int j = i+1;j < s;j++){
                if(nums[i]+nums[j]==target){
                    res.push_back(i);
                    res.push_back(j);
                }
            }
        }
        return res;
    }
};
```

- 遍历两次 HashMap
  第一次循环将数组中的值和位置放入 HashMap 中，第二次循环查看 HashMap 中有没有值为 target - nums[i] 的。时间复杂度为 O(2*n)。

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> hash;
        vector<int> res;
        for(int i = 0; i < nums.size(); ++i){
            hash[nums[i]] = i;
        }
        for(int i = 0; i < nums.size(); ++i){
            int num = target - nums[i];
            if(hash.find(num)!=hash.end() && hash[num]!=i){
                res.push_back(i);
                res.push_back(hash[num]);
                //此处不返回结果会导致解会重复，但会漏解
                //题目中没有要求输出所有解，所以解法仍成立
                return res;
            }
        }
        return res;
    }
};
```

- 遍历一次HashMap
  构造一个 HashMap，i 从 0 开始，若 target - nums[i] 不在表中，则将 nums[i] 放入表中，否则返回结果。时间复杂度为 O(n)。 

```C++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> hash;
        vector<int> res;
        for(int i = 0; i < nums.size(); ++i){
            int num = target - nums[i];
            if(hash.find(num)!=hash.end() && hash[num]!=i){
                res.push_back(hash[num]);
                res.push_back(i);
                return res;
            }
            hash[nums[i]] = i;
        }
        return res;
    }
};
```

## No.2 [两数相加](https://leetcode.com/problems/add-two-numbers)

### 题目描述

给定两个非空的链表表示两个非负数，链表中的数字是反向存储的，链表中每个节点包含一个数字，要求将这两个数相加并返回链表形式的结果

### 解法

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode head(0),*temp = &head;
        int extra = 0;
        while(l1||l2||extra){
            int sum = (l1?l1->val:0)+(l2?l2->val:0)+extra;
            temp->next = new ListNode(sum%10);
            temp = temp->next;
            extra = sum/10;
            l1 = l1?l1->next:l1;
            l2 = l2?l2->next:l2;
        }
        return head.next;
    }
};
```

## No.3 [最长无重复字符子串](https://leetcode.com/problems/longest-substring-without-repeating-characters)

### 题目描述

给定一个字符串 s，找出 s 中无重复字符的最长子串。

### 解法

- 暴力搜索
  从每个字符开始寻找不含重复字符的子串，找出最大长度。时间复杂度为 O(n^2)

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int maxL = 0, ch[128];
        for(int i = 0;i < s.length()-maxL;i++){
            int len = 1;
            memset(ch, 0 ,sizeof(ch));
            ch[s[i]] = 1;
            for(int j = i+1;j < s.length();j++){
                if(ch[s[j]])
                    break;
                else{
                    len++;
                    ch[s[j]] = 1;
                } 
            }
            maxL = max(maxL, len);
        }
        return maxL;
    }
};
```

- 滑动窗口
  定义两个变量 i、j，i 表示子串从第 i 个字符开始，j 表示当前搜索到第 j 个字符，Sij 表示 S 从第 i 个字符到第 j - 1 个字符的子串。从第 i 个字符开始，如果 S[j] 在 Sij 中出现两次，则 i 向前一步；否则，j 向前一步，并**计算结果(考虑 S 长度为1的情况)**。时间复杂度为 O(2*n)。

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        vector<int> dict(256, 0);
        int res = 0, i = 0, j = 0;
        while(i<s.length() && j<s.length()){
            if(!dict[s[j]]){
                dict[s[j++]]++;
                res = max(res, j-i);
            }
            else{
                dict[s[i++]] = 0;
            }
        }
        return res;
    }
};
```

- 滑动窗口改进版
  Sij 表示字符串 S 从第 i 个字符到第 j - 1 个字符的子串，定义一个 dict 数组存放到目前为止 S[i] 出现的最后位置。start 表示**不包含重复字符的子串的起始位置 - 1**。从 S 的第 0 个字符开始，如果该字符未出现过，则 `dict[s[i]] = -1`，修改该字符最后出现位置为 i，并计算结果；若该字符在子串 Sij 中未出现过，则 `dict[s[i]] < start`；若该字符在 Sij 中重复，则 `dict[s[i]] > start`，于是 `start = dict[s[i]]`，将 start 赋值为重复字符最后出现的位置。

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        vector<int> dict(256, -1);
        int res = 0, start = -1;
        for(int i = 0; i < s.length(); i++){
            if(dict[s[i]] > start)
                start = dict[s[i]];
            dict[s[i]] = i;
            res = max(res, i-start);
        }
        return res;
    }
};
```

## No.4 [两个有序数组的中位数](https://leetcode.com/problems/median-of-two-sorted-arrays)

### 题目描述

给定两个有序数组，找出这两个数组的中位数(假定这两个数组都是非空的)

### 解法

一个数组中中位数具有如下性质：

- 中位数左侧数字的数量等于右侧数字的数量
- 中位数左侧的数都小于右侧的数

设 nums1 的长度为 m，nums2 的长度为 n，将 nums1 从 i 处分开，将 nums2 从 j 处分开，将 nums1 左侧和 nums2 右侧拼成一个数组 left，将 nums1 右侧和 nums2 右侧拼成一个数组 right；若有 `i+j==m+n-i-j(i+j==m+n+1-i-j) && max(left)<=min(right)`，则表明当前分法已满足上述性质，即已找到中位数。那么现在问题就变成如何找到合适的 i 和 j；i 和 j 满足`i+j==m+n-i-j(i+j==m+n+1-i-j)` 条件，那么只要使 `max(left)<=min(right)` 即可，这个条件可转化为 `nums1[i-1]<=nums2[j] && nums2[j-1]<=nums1[i]`。当 `nums1[i-1]>nums2[j]` 时，说明 i 较大，需要向左移动；当 `nums2[j-1]>nums1[i]`，说明 i 较小，需要向右移动。

注意点：
1. j = (m+n+1)/2 - i，为避免 j < 0 的情况，应使 `nums1.size() < nums2.size()`。
2. 只需要找到 i，j 就确定了，因此只需要在较短的数组中搜索即可，为加快搜索速度，可采用二分搜索。
3. 边界条件：

 - 当 i == 0 时，max(left) 为nums2[j-1]。
 - 当 j == 0 时，max(left) 为nums1[i-1]。
 - 当 i == m 时，min(right) 为nums2[j]。
 - 当 j == n 时，min(right) 为nums1[i]。

若 m + n 为奇数，则中位数为 `max(nums1[i-1],nums2[j-1])`；若 m + n 为偶数，则中位数为 `(max(nums1[i-1], nums2[j-1])+min(nums1[i], nums2[j]))/2.0`。

```C++
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int m = nums1.size(), n = nums2.size();
        if(m > n) return findMedianSortedArrays(nums2, nums1);
        int left = 0, right = m;
        int tmp = (m+n+1)/2, i, j;
        while(left <= right){
            i = (left+right)/2;
            j = tmp-i;
            if(i>left && nums1[i-1]>nums2[j]){
                right = i - 1;
            }
            else if(i<right && nums2[j-1]>nums1[i]){
                left = i + 1;
            }
            else{
                int maxLeft = 0, minRight = 0;
                if(i == 0) maxLeft = nums2[j-1];
                else if(j == 0) maxLeft = nums1[i-1];
                else maxLeft = max(nums1[i-1], nums2[j-1]);
                
                if(i == m) minRight = nums2[j];
                else if(j == n) minRight = nums1[i];
                else minRight = min(nums1[i], nums2[j]);
                if((m+n)%2) return maxLeft;
                else return (maxLeft + minRight)/2.0;
            }
        }
        return 0;
    }
};
```

## No.5 [最长回文子串](https://leetcode.com/problems/longest-palindromic-substring)

### 题目描述

给定一个字符串 s，找出 s  中最长的回文子串(假设 s 的长度不超过1000)

### 解法

- [常规解法](https://leetcode.com/problems/longest-palindromic-substring/solution/)：
  常规解法有如下三种：
    - 将字符串 S 反转为 S'，寻找 S 与 S' 的[最长公共子串](https://en.wikipedia.org/wiki/Longest_common_substring_problem)
    - 暴力搜索字符串 S 的每个子串，判断是否是回文子串
    - 将 S 的每个字符向左右扩展，判断是否是回文子串

    方法一需注意**找到最长公共子串后仍需判断该子串是否是回文子串**
    方法二的时间复杂度为 O(n^3)
    这里我们重点讨论方法三：
    回文子串可能存在两种情况，即 `aba` 型和 `abba` 型。
    无论哪种类型，都是从中心开始扩展，寻找最大扩展长度。
    代码如下：

```C++
class Solution {
public:
    string longestPalindrome(string s) {
        if(s.length()<1) return "";
        int len, maxLen = 0, start = 0;
        for(int i = 0; i < s.length(); i++){
            len = max(findMaxLen(s, i, i)*2-1, findMaxLen(s, i, i+1)*2);
            if(len > maxLen){
                start = i-(len-1)/2;
                maxLen = len;
            }
        }
        return s.substr(start, maxLen);
    }
    
    int findMaxLen(string s, int left, int right){
        int i = 0;
        while(left-i>=0 && right+i<s.length() && s[left-i]==s[right+i]){
            i++;
        }
        return i;
    }
};
```

- [O(n)解法](https://articles.leetcode.com/longest-palindromic-substring-part-ii/)：
  为了方便理解，首先先将字符串 S 进行预处理为 T，T的长度为奇数。
  然后构造一个长度与 T 长度相等的数组 P，其中 P[i] 指以 i 为中心的回文字符串长度。
  P 数组的构造过程如下：
  首先定义两个变量 C 和 R，C 表示回文串的中心，R 表示回文串的右边界 。P[i] 的构造公式为 `P[i]=(R>i)?min(R-i,P[i_mirror]):0`，i_mirror 表示 i 关于 C 的对称点。理解如下：
  当 R <= i 即 i 不在回文串中时，由于还没有搜索过，于是赋 0 ；当 R > i 时，表明 i 在回文串中，若 `R - i > P[i_mirror]`，则表明以i\_mirror为中心的回文串包裹在以 C 为中心的回文串中，根据对称的性质，`P[i] = P[i_mirror]`；若  ` R - i < P[i_mirror]`，则表明以 i\_mirror 为中心的回文串与以 C 为中心的回文串有重叠的部分，根据对称性质，可以保证以 i 为中心的回文串到边界 R 的部分是对称的，再往右就需要继续搜索， 即 `P[i] = R - i`，综上，当 R > i 时，`P[i] = min(R-i, P[i_mirro r])`。
  代码如下：

```C++     
// Transform S into T.
// For example, S = "abba", T = "^#a#b#b#a#$".
// ^ and $ signs are sentinels appended to each end to avoid           bounds checking.
string preProcess(string s) {
    int n = s.length();
    if (n == 0) return "^$";
    string ret = "^";
    for (int i = 0; i < n; i++)
        ret += "#" + s.substr(i, 1);
    ret += "#$";
    return ret;
}
 
string longestPalindrome(string s) {
    string T = preProcess(s);
    int n = T.length();
    int *P = new int[n];
    int C = 0, R = 0;
    for (int i = 1; i < n-1; i++) {
        int i_mirror = 2*C-i; // equals to i' = C - (i-C)
        P[i] = (R > i) ? min(R-i, P[i_mirror]) : 0;
        // Attempt to expand palindrome centered at i
        while (T[i + 1 + P[i]] == T[i - 1 - P[i]])
            P[i]++;
        // If palindrome centered at i expand past R,
        // adjust center based on expanded palindrome.
        if (i + P[i] > R) {
            C = i;
            R = i + P[i];
        }
    }
    // Find the maximum element in P.
    int maxLen = 0;
    int centerIndex = 0;
    for (int i = 1; i < n-1; i++) {
        if (P[i] > maxLen) {
            maxLen = P[i];
            centerIndex = i;
        }
    }
    delete[] P;
    return s.substr((centerIndex - 1 - maxLen)/2, maxLen);
}
```

## No.6 [之字形变换](https://leetcode.com/problems/zigzag-conversion/)

### 题目描述

给定一个字符串 s 和行数 numsRow，将字符串转换成之字形格式并输出该格式下的新字符串。

### 解法

- 我的解法

```C++
class Solution {
public:
    string convert(string s, int numRows) {
        if(s=="") return "";
        if(numRows==1) return s;
        string res = "";
        char a[numRows][s.length()];
        memset(a,0,sizeof(a));
        int dirs[4][2] = {{-1,0},{1,0},{1,1},{-1,1}};
        int r = 0, c = 0;
        enum State{UP, DOWN, BOTTOM_RIGHT, TOP_RIGHT};
        enum State state = DOWN;
        for(int i = 0;i < s.length();i++){
            a[r][c] = s[i];
            switch(state){
                case DOWN:
                    if(r==numRows-1){
                        state = TOP_RIGHT;
                        r+=dirs[3][0];
                        c+=dirs[3][1];
                    }
                    else r+=dirs[1][0];
                    break;
                case UP:
                    if(r==0){
                        state = BOTTOM_RIGHT;
                        r+=dirs[2][0];
                        c+=dirs[2][1];
                    }
                    else r+=dirs[2][0];
                    break;
                case TOP_RIGHT:
                    if(r==0){
                        state = DOWN;
                        r+=dirs[1][0];
                    }
                    else{
                        r+=dirs[3][0];
                        c+=dirs[3][1];
                    } 
                    break;
                case BOTTOM_RIGHT:
                    if(r==numRows-1){
                        state = UP;
                        r+=dirs[0][0];
                    }
                    else{
                        r+=dirs[2][0];
                        c+=dirs[2][1];
                    } 
                    break;
            }
        }
        for(int i = 0;i < numRows;i++){
            for(int j = 0;j < s.length();j++){
                if(a[i][j]!=0) res += a[i][j];
            }
        }
        return res;
    }
};
```

- 大佬解法
  要求效果虽然是之字形，但其实可以当作一个蛇型矩阵。

```C++
string convert(string s, int nRows) {
    if (nRows <= 1)
        return s;
    const int len = (int)s.length();
    string *str = new string[nRows];
    int row = 0, step = 1;
    for (int i = 0; i < len; ++i){
        str[row].push_back(s[i]);
        if (row == 0)
            step = 1;
        else if (row == nRows - 1)
            step = -1;
        row += step;
    }
    s.clear();
    for (int j = 0; j < nRows; ++j){
        s.append(str[j]);
    }
    delete[] str;
    return s;
}
```

## No.7 [翻转数字](https://leetcode.com/problems/reverse-integer/)

### 题目描述

给一个32位有符号整数，将其数字进行翻转

注意点：**溢出问题！**

### 解法

```C++
class Solution {
public:
    int reverse(int x) {
        int result = 0;
        while(x){
            result = result*10 + x%10;
            if(result%10 != x%10)
                return 0;
            x /= 10;
        }
        return result;
    }
};
```

## No.8 [字符串转数字](https://leetcode.com/problems/string-to-integer-atoi/)

### 题目描述

给定一个字符串 s，将其转换为数字。

注意点：

- 字符串中可能会包含空格
- 数字前可能会有+/-
- 如果字符串不包含数字或第一个非空格字符不是数字，则返回 0
- 假设只处理32位有符号整数的情况，如果越界则返回 `INT_MAX` 或 ` INT_MAX`

### 解法

```C++
class Solution {
public:
    int myAtoi(string str) {
        int i = 0;int res = 0;
        bool isNegative = false;
        for(;str[i]==' ';++i);
        if(str[i]=='-'){
            isNegative = true;
            i++;
        } 
        else if(str[i]=='+') ++i;
        if(isdigit(str[i])){
            while(isdigit(str[i])){
                res = res*10 + (str[i]-'0');
                if(res%10 != (str[i]-'0'))
                    return isNegative?INT_MIN:INT_MAX;
                i++;
            }
            return isNegative?-1*res:res;
        }
        else return res;
    }
};
```

## No.9 [判断回文数](https://leetcode.com/problems/palindrome-number/)

### 题目描述

判断一个数是否是回文数。若该数为负数或以 '0' 结尾，则显然不是回文数。

### 解法

```C++
class Solution {
public:
    bool isPalindrome(int x) {
        if(x < 0) return false;
        int a = 0, b = x;
        while(b){
            a = a*10 + b%10;
            b /= 10;
        }
        if(x == a){
            return true;
        }
        else{
            return false;
        }
    }
};
```

## No.10 [正则表达式匹配](https://leetcode.com/problems/regular-expression-matching) 

### 题目描述

给定一个字符串 s 和一个匹配模式 p，s 和 p 中只包含字母 a-z，模式 p 中可能包含 `.` 和 `*`，`.` 能够匹配任意字符，`*` 能够匹配前一字符 0 次或多次。判断 s 和 p 是否匹配。

### 解法

- 递归解法
    首先可以确定的是 `*` 肯定不会是第一个元素，如果模式 p 为空，则 s 为空时为真，不空时为假；然后匹配第一个元素，若 s 为空，则返回 false，当 s 和 p 的第一个元素相等或 p 的第一个元素为 `.` 时匹配成功；**如果 p 的长度大于1且第二个元素为 `*` 时**，看 s 从第二个元素开始的子串与 p 是否匹配(第一个元素可能匹配多次)或 s 与模式从第三个元素开始的子串是否匹配(p 的第一个字符匹配 0 次)，显然前者第一个元素必须匹配成功；如果不满足上述条件且第一个元素匹配成功，则匹配下一元素，递归执行即可得到结果。
    时间复杂度较高。

```C++
class Solution {
public:
    bool isMatch(string s, string p) {
        int ls = s.length(), lp = p.length();
        if(!lp) return !ls;
        bool first_match = (ls && ((p[0]==s[0]) || p[0]=='.'));
        if(lp>=2 && p[1]=='*')
            return (isMatch(s, p.substr(2, lp-2)) || 
                (first_match && isMatch(s.substr(1,ls-1),p)));
        else
            return first_match && isMatch(s.substr(1,ls-1),p.substr(1,lp-1));
    }
};
```

- DP解法
  自上而下
  若用 dp[i][j] 来表示 s[0...i-1] 和 p[0...j-1]是否匹配(兼顾 s 或 p  为空的情况)，则可得到以下递推公式：

1. 当 `p[j-1]!='*'`时，若满足 `p[j-1]==s[i-1]` 或 `p[j-1]=='.'`，  则 dp[i][j] 取决于前面是否匹配，即 `dp[i][j] = dp[i-1][j-1]`
2. 当 `p[j-1]=='*'` 时，`dp[i][j] == dp[i][j-2]` 表示 p[j-2] 匹配 0 次；`dp[i][j] = dp[i-1][j] && (s[i-1]==p[j-2] || p[j-2]=='.')` 表示 p[j-2] 匹配至少 1 次。

  注意点：i 应从 0 开始，以匹配 s 为空串且 p 中每个字符匹配 0 次。

```C++
class Solution {
public:
    bool isMatch(string s, string p) {
        int m = s.length(), n = p.length(); 
        vector<vector<bool> > dp(m + 1, vector<bool> (n + 1, false));
        dp[0][0] = true;
        for (int i = 0; i <= m; i++)
            for (int j = 1; j <= n; j++)
                if (p[j - 1] == '*')
                    dp[i][j] = dp[i][j - 2] || (i > 0 && (s[i - 1] == p[j - 2] || p[j - 2] == '.') && dp[i - 1][j]);
                else dp[i][j] = i > 0 && dp[i - 1][j - 1] && (s[i - 1] == p[j - 1] || p[j - 1] == '.');
        return dp[m][n];
    }
};
```

另外还可以采用自下而上的 DP 和 NFA 的解法，此处不作解释。

## No.11 [装水最多的容器](https://leetcode.com/problems/container-with-most-water)

### 题目描述

给定 n 个非负整数 a1,a2,...,an，(i, ai)表示坐标轴上的一点，从该点作垂线垂直于x轴，两条线段可组成一个容器，求组成容器的最大容积(不可以倾斜容器且 n>=2)

### 解法

- 暴力搜索
  时间复杂度为O(n^2)，可能会超时。

```C++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int max = 0;
        if(height.size()<2) return max;
        int area = 0;
        for(int i = 0;i < height.size()-1;i++){
            for(int j = i+1;j < height.size();j++){
                if(height[i]>=height[j]) area = height[j]*(j-i);
                else area = height[i]*(j-i);
                if(area>max) max = area;
            }
        }
        return max;
    }
};
```

- 线性搜索
  贪心算法，从数组的两端开始寻找，保留较长的边。时间复杂度为O(n)

```C++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int mx = 0;
        if(height.size()<2) return mx;
        int l = 0, r = height.size()-1;
        while(l<r){
            mx = max(mx,min(height[l],height[r])*(r-l));
            height[l]>height[r]?r--:l++;
        }
        return mx;
    }
};
```

## No.12 [整数变罗马数字](https://leetcode.com/problems/integer-to-roman/)

### 题目描述

罗马数字由七个不同的符号组成：I V X L C D M，分别表示 1 5 10 50 100 500 1000，IV IX XL XC CD CM 分别表示 4 9 40 90 400 900。给定一个整数(1-3999)，输出对应的罗马数字。

### 解法

由于罗马数字表示有范围限制，所以直接枚举就可以了。

```C++
class Solution {
public:
    string intToRoman(int num) {
        if(num<1 || num>3999) return "";
        string s1[] = {"","M","MM","MMM"};
        string s2[] = {"","C","CC","CCC","CD","D","DC","DCC","DCCC","CM",};
        string s3[] = {"","X","XX","XXX","XL","L","LX","LXX","LXXX","XC"};
        string s4[] = {"","I","II","III","IV","V","VI","VII","VIII","IX"};
        return s1[num/1000]+s2[(num%1000)/100]+s3[(num%100)/10]+s4[num%10];
    }
};
```

## No.13 [罗马数字变整数](https://leetcode.com/problems/roman-to-integer/)

### 题目描述

转换规则同上一题，罗马数字范围为 1-3000

### 解法

- 从头开始遍历字符串，考虑每一种情况。

```C++
class Solution {
public:
    int romanToInt(string s) {
        if(s.length()==0) return 0;
        int res = 0;
        for(int i = 0;i < s.length();i++){
            if(s[i]=='M') res+=1000;
            else if(s[i]=='D') res+=500;
            else if(s[i]=='C'){
                if(s[i+1]=='D'){
                    res+=400;i++;
                }
                else if(s[i+1]=='M'){
                    res+=900;i++;
                }
                else res+=100;
            }
            else if(s[i]=='L') res+=50;
            else if(s[i]=='X'){
                if(s[i+1]=='L'){
                    res+=40;i++;
                }
                else if(s[i+1]=='C'){
                    res+=90;i++;
                }
                else res+=10;
            }
            else if(s[i]=='V') res+=5;
            else{
                if(s[i+1]=='V'){
                    res+=4;i++;
                }
                else if(s[i+1]=='X'){
                    res+=9;i++;
                }
                else res+=1;
            }
        }
        return res;
    }
};
```

- 从字符串末尾向前扫描，若前一个罗马字符代表的数字比当前位置的小，则表示 4 或 9 的情况，减去当前罗马字符代表的数字，否则加上该数字，解法非常巧妙。

```C++
class Solution {
public:
    int romanToInt(string s){
        unordered_map<char, int> T = { { 'I' , 1 },
                                       { 'V' , 5 },
                                       { 'X' , 10 },
                                       { 'L' , 50 },
                                       { 'C' , 100 },
                                       { 'D' , 500 },
                                       { 'M' , 1000 } };
        int sum = T[s.back()];
        for(int i = s.length() - 2; i >= 0; --i){
            if (T[s[i]] < T[s[i + 1]]){
                sum -= T[s[i]];
            }
            else{
                sum += T[s[i]];
            }
        }
        return sum;
    }
}
```

## No.14 [最长公共前缀](https://leetcode.com/problems/longest-common-prefix/)

### 题目描述

在给定的字符串数组中找出最长的公共前缀子串，若没有则返回空串

### 解法

用二重循环比较即可，时间复杂度为 O(n^2)。

```C++
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        string res = "";
        if(strs.empty()) return res;
        char ch; int j;
        for(int i = 0;i < strs[0].length();i++){
            ch = strs[0][i];
            for(j = 0;j < strs.size();j++){
                if(i>=strs[j].size()) break;
                if(strs[j][i]!=ch) break;
            }
            if(j>=strs.size()) res += ch;
            else break;
        }
        return res;
    }
};
```

此题还有其他解法，但思路与上述解法类似，故不再重复解释。

## No.15 [三数之和](https://leetcode.com/problems/3sum/)

### 题目描述

给定一个整数数组 nums，其中是否存在三个数 a b c 使 a + b + c = 0？试找出所有和为 0 的不同的数字组合

### 解法

为解题方便，应先将所给数组进行排序。然后先选定第一个数 nums[i]，然后从 i+1 和nums.size()-1 处开始向中间寻找符合题意的数，找到后插入到结果中，然后继续向中间寻找，直至遍历完整个数组。时间复杂度为 O(n^2)。

注：为避免重复，每当确定一组结果后继续遍历时都应当跳过重复元素。

```C++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> res;
        std::sort(nums.begin(), nums.end());
        for(int i = 0; i < nums.size(); ++i){
            int target = -nums[i], low = i+1, high = nums.size()-1;
            while(low < high){
                int sum = nums[low] + nums[high];
                if(sum < target) low++;
                else if(sum > target) high--;
                else{
                    vector<int> tmp(3, 0);
                    tmp[0] = nums[i];
                    tmp[1] = nums[low];
                    tmp[2] = nums[high];
                    res.push_back(tmp);
                    while(low<high && nums[low]==tmp[1]) low++;
                    while(low<high && nums[high]==tmp[2]) high--;
                }
            }
            while(i<nums.size()-1 && nums[i+1]==nums[i]) i++;
        }
        return res;
    }
};
```

## No.16 [最近的三数之和](https://leetcode.com/problems/3sum-closest/)

### 题目描述

给定一个整数数组 nums 和一个目标数 target，找出数组中三个数使这三个数的和与 target 最接近，返回三数之和(假设每个输入有且仅有一解)

### 解法

- 暴力破解
  遍历该数组，找出与目标数最接近的三数之和。时间复杂度为 O(n^3)。

```C++
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        int res = INT_MAX, n = nums.size(), tmp = res, sum;
        for(int i = 0; i < n; ++i){
            for(int j = i + 1; j < n; ++j){
                for(int k = j + 1; k < n; ++k){
                    sum = nums[i] + nums[j] + nums[k];
                    if(abs(sum - target) < tmp){
                        tmp = abs(sum - target);
                        res = sum;
                        if(!tmp) return res;
                    }
                }
            }
        }
        return res;
    }
};
```

- 改进版
解法：与上一题类似，先对数组进行排序，然后选定第一个数 nums[i]，再从 i+1 和 nums.size()-1 处向中间寻找和与 target 最接近的数，若和比 target 大，则最大值向左寻找，若和比 target 小，则次大值向右寻找。时间复杂度为 O(n^2)。

```C++
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        std::sort(nums.begin(), nums.end());
        int res = nums[0]+nums[1]+nums[2];
        for(int i = 0; i < nums.size()-2; ++i){
            int low = i+1, high = nums.size()-1;
            while(low < high){
                int sum = nums[i]+nums[low]+nums[high];
                if(sum == target) return target;
                else if(abs(sum-target)<abs(res-target))
                    res = sum;
                if(sum > target) high--;
                else low++;
            }
        }
        return res;
    }
};
```

## No.17 [电话号码的数字组合](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

### 题目描述

给定一个由数字 2-9 组成的字符串，输出数字对应字母的所有组合。

### 解法

- 迭代解法
遍历字符串时声明一个临时数组 tmp，将当前位置的数字对应的字母分别与已有结果 res 进行拼接，然后将 tmp 与 res 进行交换。

注：使用swap()可避免复制，执行速度更快。

```C++
class Solution {
public:
    vector<string> letterCombinations(string digits) {
        vector<string> res;
        if(!digits.size()) return res;
        res.push_back("");
        static const vector<string> v = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
        for(int i = 0; i < digits.size(); ++i){
            int num = digits[i] - '0';
            vector<string> tmp;
            for(int j = 0; j < res.size(); ++j){
                for(int k = 0; k < v[num].size(); ++k){
                    tmp.push_back(res[j]+v[num][k]);
                }
            }
            res.swap(tmp);
        }
        return res;
    }
};
```

- 回溯法 
采用深度遍历搜索，将每一个字符串添加到结果集中。

```C++
class Solution {
public:
    vector<vector<char>> v = {{}, {}, 
                              {'a', 'b', 'c'},
                              {'d', 'e', 'f'},
                              {'g', 'h', 'i'},
                              {'j', 'k', 'l'},
                              {'m', 'n', 'o'},
                              {'p', 'q', 'r', 's'},
                              {'t', 'u', 'v'},
                              {'w', 'x', 'y', 'z'}
                             };
    vector<string> letterCombinations(string digits) {
        vector<string> res;
        if(!digits.size()) return res;
        string local = "";
        foo(0, digits, res, local);
        return res;
    }
    
    void foo(int pos, string digits, vector<string> &res, string local){
        if(pos == digits.size()){
            res.push_back(local);
            return;
        } 
        char c = digits[pos] - '0';
        for(int i = 0; i < v[c].size(); ++i){
            local.push_back(v[c][i]);
            foo(pos+1, digits, res, local);
            local.pop_back();
        }
    }
    
};
```

## No.18 [四数之和](https://leetcode.com/problems/4sum/)

### 题目描述

给定一个整数数组 nums 和一个整数  target，其中是否存在四个数 a b c d 使 a + b + c + d = target？试找出所有和为 target 的不同的数字组合。

### 解法

为方便解题，先对给定数组进行排序。然后先确定两个数的位置 i 和 j，在剩下的数中从左右两边依次向中间推进。

注：为避免重复，每当确定一组结果后继续遍历时都应当跳过重复元素。确定i 和 j 时也需要跳过重复元素。 

```C++
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        int len = nums.size();
        vector<vector<int>> res;
        if(len < 4) return res;
        std::sort(nums.begin(), nums.end());
        for(int i = 0; i < len-3; ++i){
            if(i>0 && nums[i]==nums[i-1]) continue;
            if(nums[i]+nums[i+1]+nums[i+2]+nums[i+3] > target) break;
            if(nums[i]+nums[len-1]+nums[len-2]+nums[len-3] < target) continue;
            for(int j = i+1; j < len-2; ++j){
                if(j>i+1 && nums[j]==nums[j-1]) continue;
                if(nums[i]+nums[j]+nums[j+1]+nums[j+2] > target) break;
                if(nums[i]+nums[len-1]+nums[len-2]+nums[j] < target) continue;
                int left = j + 1, right = len - 1;
                while(left < right){
                    int sum = nums[i]+nums[j]+nums[left]+nums[right];
                    if(sum>target) right--;
                    else if(sum<target) left++;
                    else{
                        vector<int> tmp = {nums[i], nums[j], nums[left], nums[right]};
                        res.push_back(tmp);
                        do{left++;}while(nums[left]==nums[left-1] && left<right);
                        do{right--;}while(nums[right]==nums[right+1] && left<right);
                    }
                }
            }
        }
        return res;
    }
};
```

## No.19 [删除倒数第N个元素](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

### 题目描述

给定一个链表，删除倒数第 N 个结点并返回头结点

注意点：需要判断是否是删除头结点。

### 解法

- 首先遍历链表得到链表长度 len，就可求出需要删除元素的位置。再从头结点开始找到第 len - n 个结点，删除该结点。时间复杂度为 O(2*n)。

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        int len = 1;
        ListNode *tmp = head;
        while(tmp->next){
            len++;
            tmp = tmp->next;
        }
        int pos = len-n;
        if(pos==0) return head->next;
        tmp = head;
        while(--pos) tmp = tmp->next;
        tmp->next = tmp->next->next;
        return head;
    }
};
```

- 先用一个指针 p 向前走 n 步，再用另一个指针 q 从头开始与 p 共同向前走，当 p 走到链表结尾时 q 即为倒数第 n 个元素。时间复杂度为 O(n)。  

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode *p = head, *q = head;
        while(n--){
            p = p -> next;
        }
        if(!p) return head->next;
        while(p->next){
            p = p -> next;
            q = q -> next;
        }
        q -> next = q -> next -> next;
        return head;
    }
};
```

## No.20 [括号匹配](https://leetcode.com/problems/valid-parentheses/)

### 题目描述

给定一个字符串，其中包含`'{'`,`'}'`,`'('`,`')'`,`'['`和`']'`，判断该字符串中括号是否匹配

### 解法

可用栈来解决，遍历字符串，若遇到左括号则入栈，若遇到右括号，则与栈顶元素比较，若匹配则将栈顶元素出栈，若不匹配则返回 false，**遍历结束后若栈为空，则返回 true**。

```C++
class Solution {
public:
    bool isValid(string s) {
        stack<char> st;
        for(char c:s){
            if(c=='('||c=='{'||c=='['){
                st.push(c);
                continue;
            }
            if(st.empty()) return false;
            switch(c){
                case ')':
                    if(st.top()=='(') st.pop();
                    else return false;
                    break;
                case '}':
                    if(st.top()=='{') st.pop();
                    else return false;
                    break;
                case ']':
                    if(st.top()=='[') st.pop();
                    else return false;
                    break;
            }
        }
        return st.empty()?true:false;
    }
};
```

## No.21 [合并有序链表](https://leetcode.com/submissions/detail/156361306/)

### 题目描述

给定两个有序链表，将其合并为一个新的有序链表

### 解法

用指针 p 指向 l1 头结点，指针 q 指向 l2 头结点，比较 `p -> val` 和 `q -> val`，将较小的结点插入新链表中并指向下一个元素，当某一个链表遍历结束后将另一个链表中剩下的元素接在新链表的最后。

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode head(0), *tmp = &head;
        while(l1 && l2){
            tmp->next = new ListNode(0);
            tmp = tmp->next;
            if(l1->val < l2->val){
                tmp->val = l1->val;
                l1 = l1->next;
            }
            else{
                tmp->val = l2->val;
                l2 = l2->next;
            }
        }
        if(l1){
            tmp->next = l1;
        }
        if(l2){
            tmp->next = l2;
        }
        return head.next;
    }
};
```

## No.22 [产生合法括号对](https://leetcode.com/problems/generate-parentheses/)

### 题目描述

给出 n 组括号对，生成所有合法的括号对组合。

### 解法

- 递归回溯法
  用 m 表示左括号剩余数，n 表示右括号剩余数，通过深度优先搜索，采用递归和回溯的算法，先添加 '('，再添加 ')'，为保证生成的括号对合法，m 应该不大于 n。

```C++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> res;
        if(n==0) return res;
        string str = "";
        foo(res, n, n, str);
        return res;
    }
    void foo(vector<string> &res, int m, int n, string local){
        if(m>n || m<0) return;
        if(n==0){
            res.push_back(local);
            return ;
        }
        local.push_back('(');
        foo(res, m-1, n, local);
        local.pop_back();
        local.push_back(')');
        foo(res, m, n-1, local);
        local.pop_back();
    }
};
```

- 递归改进
  用 n 表示剩余左括号数，m 表示保证合法所需右括号数，采用递归的方法，当 `m = n = 0` 时表示生成了一组合法的括号对。

```C++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> res;
        addingpar(res, "", n, 0);
        return res;
    }
    void addingpar(vector<string> &v, string str, int n, int m){
        if(n==0 && m==0) {
            v.push_back(str);
            return;
        }
        if(m > 0){ addingpar(v, str+")", n, m-1); }
        if(n > 0){ addingpar(v, str+"(", n-1, m+1); }
    }
};
```

本题还可采用暴力破解法，此处不予详细解释。

## No.23 [合并k组有序列表](https://leetcode.com/problems/merge-k-sorted-lists/)

### 题目描述：



### 解法：

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode head(0), *tmp = &head;
        for(int i = 0; i < lists.size(); ++i){
            if(!lists[i]){
                lists.erase(lists.begin()+i);
                --i;
            }         
        }
        while(lists.size()){
            int min = INT_MAX, pos = 0;
            for(int i = 0; i < lists.size(); ++i){
                if(min > lists[i]->val){
                    min = lists[i]->val;
                    pos = i;
                }
            }   
            tmp->next = new ListNode(0);
            tmp = tmp->next;
            tmp->val = min;
            if(!lists[pos]->next){
                lists.erase(lists.begin()+pos);
                continue;
            }                
            lists[pos] = lists[pos]->next;
        }
        return head.next;
    }
};
```

## No.24 [两两交换结点](https://leetcode.com/problems/swap-nodes-in-pairs/)

### 题目描述

给定一个链表，将其中结点两两交换。

注意点：

- 空间复杂度必须为 O(1)
- 不能改变结点中元素的值

### 解法

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if(!head) return head;
        ListNode *tmp = head->next, *h = tmp?tmp:head, *fore = head;
        while(head->next){
            fore->next = head->next;
            head->next = tmp->next;
            tmp->next = head;
            fore = head;
            head = head->next;
            if(!head) break;
            tmp = head->next;
        }
        return h;
    }
};
```

## No.25 []()

### 题目描述

### 解法

```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        if(k<=1) return head;
        ListNode *tmp = head, *tail;
        for(int i = 0; i < k; i++){
            if(i==k-1) tail = tmp;
            if(tmp) tmp = tmp->next;
            else return head;    
        }
        ListNode *tmp2 = head->next, *tmp3 = tmp2->next;
        head->next = reverseKGroup(tmp, k);
        tail->next = head;
        while(tmp2!=tail){
            tmp2->next = tail->next;
            tail->next = tmp2;
            tmp2 = tmp3;
            tmp3 = tmp3->next;
        }
        return tail;
    }
};
```

## No.26 []()

### 题目描述

### 解法

```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if(nums.size()<=1) return nums.size();
        for(int i = 1; i < nums.size(); i++){
            if(nums[i-1]==nums[i]){
                nums.erase(nums.begin()+i);
                i--;
            }
        }
        return nums.size();
    }
};
```

## No.27 []()

### 题目描述

### 解法

```C++
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        for(int i = 0; i < nums.size();i++){
            if(nums[i]==val){
                nums.erase(nums.begin()+i);
                i--;
            }
        }
        return nums.size();
    }
};
```

## No.28 []()

### 题目描述

### 解法

```C++
class Solution {
public:
    int strStr(string haystack, string needle) {
        if(needle=="") return 0;
        for(int i = 0;i < haystack.size();i++){
            int j = 0, k = i;
            while(haystack[k++]==needle[j++]);
            if(j>needle.size())
                return i;
        }
        return -1;
    }
};
```

## No.29 []()

### 题目描述

### 解法

```C++
class Solution {
public:
    int divide(int dividend, int divisor) {
        if((dividend==INT_MIN&&divisor==-1)||divisor==0) return INT_MAX;
        int res = 0;
        int sign = (dividend<0)^(divisor<0)?-1:1;
        long dvd = labs(dividend);
        long dvs = labs(divisor);
        while(dvd>=dvs){
            long tmp = dvs; int cnt = 1;
            while(dvd>=(tmp<<1)){
                tmp<<=1;
                cnt<<=1;
            }
            dvd-=tmp; 
            res+=cnt;
        }
        return sign*res;
    }
};
```