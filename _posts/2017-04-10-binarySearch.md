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
