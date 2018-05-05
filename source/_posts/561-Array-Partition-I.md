---
title: 561. Array Partition I
date: 2018-05-05 15:42:00
tags: 
categories: 
- 刷题
- LeetCode
---

>Given an array of **2n** integers, your task is to group these integers into **n** pairs of integer, say (a1, b1), (a2, b2), ..., (an, bn) which makes sum of min(ai, bi) for all i from 1 to n as large as possible. 

**Example 1:** 

```
Input: [1,4,3,2]

Output: 4
Explanation: n is 2, and the maximum sum of pairs is 4 = min(1, 2) + min(3, 4).
```

**Note:** 

1. **n** is a positive integer, which is in the range of [1, 10000].
2. All the integers in the array will be in the range of [-10000, 10000].

```java
class Solution {
    public int arrayPairSum(int[] nums) {
        int res = 0;
        int[] count = new int[20001];
        for(int i : nums){
            count[i + 10000]++;
        }
        
        boolean odd = true;
        for(int i = 0; i < count.length; i++){
            while(count[i] > 0){
                if(odd){
                    res += i - 10000;
                }
                odd = !odd;
                count[i]--;
            }
        }
        
        return res;
    }
}
```

