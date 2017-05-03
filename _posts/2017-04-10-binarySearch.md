---
layout: post
title: "二分查找"
date: 2017-04-10
description: "leetcode题解，二分查找"
tag: leetcode-算法讲解
---   

### 二分查找

#### 483. Smallest Good Base

For an integer n, we call k>=2 a good base of n, if all digits of n base k are 1.

Now given a string representing n, you should return the smallest good base of n in string format.

Example 1:
Input: "13"
Output: "3"
Explanation: 13 base 3 is 111.
Example 2:
Input: "4681"
Output: "8"
Explanation: 4681 base 8 is 11111.
Example 3:
Input: "1000000000000000000"
Output: "999999999999999999"
Explanation: 1000000000000000000 base 999999999999999999 is 11.

这个题给定一个n, 找到一个最小基数k, 使得n转为k进制后 各个位都是1.
主要思路： k最小， 那么转为k进制后的位数也最长， 所以可以从最多位数的数字开始计算，显然最长的时候就是k=2的时候， 即log(n + 1) / log(2)位， 比如n=3, 转为二进制最多有2位。最短的时候长度为2，此时k 就是n-1, 转换后为11。比如上面的example3。然后看看有没有合适的k，使得k进制计算后等于数字n. 寻找k的时候就可以借助二分查找。同理找k的时候，最小是2， 最大是 pow(num, 1.0 / (p - 1)); （要保证 k^p-1 + k ^ p-2... + k^0 >= num, 再宽松一点，k^p-1 >= num)

```c++
class Solution {
public:
    string smallestGoodBase(string n) {
        typedef unsigned long long ll;
        ll num = stol(n);
        for(ll p = log(num + 1) / log(2) ; p >= 2;  --p){
            ll lk = 2, rk = pow(num, 1.0 / (p - 1))+ 1;
            while(lk <= rk){
                ll mk = lk + (rk - lk) / 2;
                ll sum = 0;
                for(ll i = 0; i < p; i++){
                    sum = sum * mk + 1;
                }
                if(sum > num){
                    rk = mk - 1;
                }else if(sum < num){
                    lk = mk + 1;
                }else{
                    return to_string(mk);
                }
            }
        }
        return to_string(num -1);
    }
};
```

### 475. Heaters

Winter is coming! Your first job during the contest is to design a standard heater with fixed warm radius to warm all the houses.

Now, you are given positions of houses and heaters on a horizontal line, find out minimum radius of heaters so that all houses could be covered by those heaters.

So, your input will be the positions of houses and heaters seperately, and your expected output will be the minimum radius standard of heaters.

Note:
Numbers of houses and heaters you are given are non-negative and will not exceed 25000.
Positions of houses and heaters you are given are non-negative and will not exceed 10^9.
As long as a house is in the heaters' warm radius range, it can be warmed.
All the heaters follow your radius standard and the warm radius will the same.
Example 1:
Input: [1,2,3],[2]
Output: 1
Explanation: The only heater was placed in the position 2, and if we use the radius 1 standard, then all the houses can be warmed.
Example 2:
Input: [1,2,3,4],[1,4]
Output: 1
Explanation: The two heater was placed in the position 1 and 4. We need to use radius 1 standard, then all the houses can be warmed.

题目大意： 水平线上一组 location代表房子,  然后下一个vector 代表有火的位置， 求火的温暖半径至少是多少才能将每个房子都温暖过来。

对于一个房子，我们需要看他左边的火 和 右边的火 哪个更近

* 解法一：

```c++
class Solution {
public:
    int findRadius(vector<int>& houses, vector<int>& heaters) {
        sort(houses.begin(), houses.end());
        sort(heaters.begin(), heaters.end());
        int max = 0;
        int heaterIndex = 0;
        for(int i = 0; i < houses.size(); i++){
            //一个点，若在两个heater之间，就选一个距离最近的
           while(heaterIndex + 1 < heaters.size() && (abs(heaters[heaterIndex + 1] - houses[i])) <= abs(heaters[heaterIndex] - houses[i])){
               heaterIndex++;
           }
           if(abs(heaters[heaterIndex] - houses[i]) > max){
               max = abs(heaters[heaterIndex] - houses[i]);
           }
        }
        return max;
    }
};
```
* 解法二

```c++
#define rad(ht, hs) ht==-1?INT_MAX: abs(hs - ht)
class Solution {
public:
    int findRadius(vector<int>& houses, vector<int>& heaters) {
        //这个方法的大致思路，不用对house 排序了， 直接对每个house,
        //在排序好的 heaters 中找到比他略大 和比他略小的两个position.
        sort(heaters.begin(), heaters.end());
        int maxR = 0;
        for(int i = 0; i < houses.size(); i++){
            int left = searchSmaller(heaters, houses[i]);
            int right = searchBigger(heaters, houses[i]);
            int radius = min (rad(left, houses[i]), rad(right, houses[i]));
            maxR = max(maxR,radius);
        }
        return maxR;
    }

    int searchSmaller(vector<int>&heaters, int x){
        int left = 0, right = heaters.size() - 1;
        while(left < right){
            int mid = (left + right) / 2;
            if(heaters[mid] <= x){
                if(heaters[mid] == x  ||  (mid + 1 >= heaters.size()) || heaters[mid + 1] > x){
                    return heaters[mid];
                }
                left = mid + 1;
            }else{
                right = mid - 1;
            }
        }
        return heaters[left];
    }

    int searchBigger(vector<int>&heaters, int x){
        int left = 0, right = heaters.size() - 1;
        while(left < right){
            int mid = (left + right) / 2;
            if(heaters[mid] >= x){
                if(heaters[mid] == x  ||  (mid - 1 < 0) || heaters[mid - 1] < x){
                    return heaters[mid];
                }
                right = mid - 1;
            }else{
                left = mid + 1;
            }
        }
        return heaters[left];
    }
};
```

#### 454. 4Sum II
Given four lists A, B, C, D of integer values, compute how many tuples (i, j, k, l) there are such that A[i] + B[j] + C[k] + D[l] is zero.

To make problem a bit easier, all A, B, C, D have same length of N where 0 ≤ N ≤ 500. All integers are in the range of -228 to 228 - 1 and the result is guaranteed to be at most 231 - 1.

思路：如果直接枚举的话， n^4， 时间复杂度太大。 我们可以任意地两两组织一下，题目告诉了ABCD数组的大小一样
。此题需要注意 重复情况。

解法1： 用hash, 讲A+B 哈希存储起来， 然后查找 -（C + D）

```c++
class Solution {
public:
    int fourSumCount(vector<int>& A, vector<int>& B, vector<int>& C, vector<int>& D) {
        unordered_map<int, int> map;
        for(int i = 0; i < A.size(); i++){
            for(int j = 0; j < B.size(); j++){
                map[A[i] + B[j]] ++;
            }
        }
        int r = 0;
        for(int i = 0; i < C.size(); i++){
            for(int j = 0; j < D.size(); j++){
                unordered_map<int, int>::iterator it = map.find(-C[i] - D[j]);
                if(it != map.end()){
                    r += it->second;
                }
            }
        }
        return r;
    }
};
```

解法2： 将 A* B， C* D 存储起来，排序， 遍历AB, 在CD 中 折半查找  

```c++
class Solution {
public:
    int low(vector<int> & CD, int num){
        int left = 0;
        int right = CD.size() - 1;
        while(left <= right){
            int mid = (left + right) / 2;
            if(CD[mid] < num){
                left = mid + 1;
            }else{
               right = mid - 1;
            }
        }
        return CD[left]==num? left : -1;
    }

    int high(vector<int>& CD, int num){
        int left = 0;
        int right = CD.size() - 1;
        while(left <= right){
            int mid = (left + right) / 2;
            if(CD[mid] <= num){
                left = mid + 1;
            }else{
               right = mid - 1;
            }
        }
        return CD[right]==num? right : -1;
    }

    int fourSumCount(vector<int>& A, vector<int>& B, vector<int>& C, vector<int>& D) {
        vector<int> AB;
        for(int i = 0; i < A.size(); i++){
            for(int j = 0 ; j < B.size(); j++){
                AB.push_back(A[i] + B[j]);
            }
        }
         vector<int> CD;
         for(int i = 0; i < C.size(); i++){
            for(int j = 0 ; j < D.size(); j++){
                CD.push_back(C[i] + D[j]);
            }
        }
        sort(AB.begin(), AB.end());
        sort(CD.begin(), CD.end());
        int count = 0;

        for(int i = 0; i < AB.size(); i++){
            int num = - AB[i];
            int l = low(CD, num);
            if(l != -1){
                int r = high(CD, num);
                count += (r- l + 1);
            }
        }
        return count;
    }
};
```

#### 441
You have a total of n coins that you want to form in a staircase shape, where every k-th row must have exactly k coins.

Given n, find the total number of full staircase rows that can be formed.

n is a non-negative integer and fits within the range of a 32-bit signed integer.

Example 1:

n = 5

The coins can form the following rows:
¤
¤ ¤
¤ ¤

Because the 3rd row is incomplete, we return 2.

思路： 这个题可以利用等差数列 做个不等式，然后二分查找，当然也可以利用求根公式直接求出来。 二分查找的时候，一定注意不要越界。 因为我们用到了 2 * n,  是有可能超出整数范围的。

```c++
class Solution {
public:
    int arrangeCoins(int n) {
        if(n < 1){
            return 0;
        }
        long nn = (long) n;
        long kl = 0;

        long kr = sqrt( 2 * nn);

        while(kl <= kr){
            long mid = (kl + kr) / 2;
            if(mid * (mid + 1) <= 2 * nn){
               kl = mid + 1;
            }else{
               kr = mid -1;
            }
        }
        return kr;
    }
};
```

#### 35. Search Insert Position
Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You may assume no duplicates in the array.

Here are few examples.
[1,3,5,6], 5 → 2

[1,3,5,6], 2 → 1

[1,3,5,6], 7 → 4

[1,3,5,6], 0 → 0

思路： 此题和找 low bound 差不多的算法

```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() -1;
        while(left <= right){
            int mid =  (left + right) / 2;
            if(nums[mid] < target){
                left = mid + 1;
            }else
                right = mid - 1;

        }
        return left;
    }
};
```



#### 436. Find Right Interval Add to List

Given a set of intervals, for each of the interval i, check if there exists an interval j whose start point is bigger than or equal to the end point of the interval i, which can be called that j is on the "right" of i.

For any interval i, you need to store the minimum interval j's index, which means that the interval j has the minimum start point to build the "right" relationship for interval i. If the interval j doesn't exist, store -1 for the interval i. Finally, you need output the stored value of each interval as an array.

Note:
You may assume the interval's end point is always bigger than its start point.
You may assume none of these intervals have the same start point.
Example 1:
Input: [ [1,2] ]

Output: [-1]

Explanation: There is only one interval in the collection, so it outputs -1.
Example 2:
Input: [ [3,4], [2,3], [1,2] ]

Output: [-1, 0, 1]

Explanation: There is no satisfied "right" interval for [3,4].
For [2,3], the interval [3,4] has minimum-"right" start point;
For [1,2], the interval [2,3] has minimum-"right" start point.

思路： 对start 排序， 然后遍历interval, 在排序好的start里找 low bound. 在排序之前要记录好原来的index

```c++
/**
 * Definition for an interval.
 * struct Interval {
 *     int start;
 *     int end;
 *     Interval() : start(0), end(0) {}
 *     Interval(int s, int e) : start(s), end(e) {}
 * };
 */

class Solution {
public:
vector<int> findRightInterval(vector<Interval>& intervals) {
    if(intervals.empty()) return {};
    int n = intervals.size();
    vector<int> res(n);
    unordered_map<int, int> mp;

    for(int i = 0; i < n; ++i){mp[intervals[i].start] = i;}

    sort(intervals.begin(), intervals.end(), [](Interval a, Interval b){
        return a.start < b.start;}
    );

    for(int i = 0; i < n; ++i){
        int k = mp[intervals[i].start];// the previous index
        int target = intervals[i].end;
        // binary search:
        if(target <= intervals[n-1].start){
       //Because we have sorted the intervals, so the start point of binary search should be i, no need to search for range [0, i-1].
            int l = i, r = n - 1;
            while(l <= r){
                int m = (l + r)/2;
                if(intervals[m].start < target){l = m + 1;}
                else if(intervals[m].start >= target){r = m - 1;}
            }
            res[k] = mp[intervals[l].start];
        }else{
            res[k] = -1;
        }
    }
    return res;
}
};
```

另外， c++ map 自己实现了 lower_bound

```c++
class Solution {
public:
    vector<int> findRightInterval(vector<Interval>& intervals) {
        map<int, int> hash;
        vector<int> res;
        int n = intervals.size();
        for (int i = 0; i < n; ++i)
            hash[intervals[i].start] = i;
        for (auto in : intervals) {
            auto itr = hash.lower_bound(in.end);
            if (itr == hash.end()) res.push_back(-1);
            else res.push_back(itr->second);
        }
        return res;
    }
};
```

#### 410. Split Array Largest Sum

Given an array which consists of non-negative integers and an integer m, you can split the array into m non-empty continuous subarrays. Write an algorithm to minimize the largest sum among these m subarrays.

Note:
If n is the length of array, assume the following constraints are satisfied:

1 ≤ n ≤ 1000
1 ≤ m ≤ min(50, n)
Examples:

Input:
nums = [7,2,5,10,8]
m = 2

Output:
18

Explanation:
There are four ways to split nums into two subarrays.
The best way is to split it into [7,2,5] and [10,8],
where the largest sum among the two subarrays is only 18.

思路： 这个题是cut数组, 找出分割成m份的最小最大sum. 所以数组子集的顺序不能改变的。（如果可以改变顺序， 是否能从大到小分？） 找出最小可能的sum, 和最大可能的 sum, 二分查找。

```c++
class Solution {
public:
    int splitArray(vector<int>& nums, int m) {
        long long left = 0, right = 0;
        for (int i = 0; i < nums.size(); ++i) {
            left = max((int)left, nums[i]);
            right += nums[i];
        }
        while (left < right) {
            long long mid = left + (right - left) / 2;
            if (can_split(nums, m, mid)) right = mid;
            else left = mid + 1;
        }
        return left;
    }
    bool can_split(vector<int>& nums, int m, int sum) {
        int cnt = 1, curSum = 0;
        for (int i = 0; i < nums.size(); ++i) {
            curSum += nums[i];
            if (curSum > sum) {
                curSum = nums[i];
                ++cnt;
                if (cnt > m) return false;
            }
        }
        return true;
    }
};
```
