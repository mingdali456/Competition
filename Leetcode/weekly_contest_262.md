# Weekly Contest 262
Chinese site ranking: 44  
Global site ranking: 75
## [5894. Two Out of Three](https://leetcode-cn.com/contest/weekly-contest-262/problems/two-out-of-three/)
We simply need to use a dict to capture the overlapped elements. Note that each array may contain duplicates. 
This is why I transform the list into a set before updating the dict.
```
class Solution(object):
    def twoOutOfThree(self, nums1, nums2, nums3):
        """
        :type nums1: List[int]
        :type nums2: List[int]
        :type nums3: List[int]
        :rtype: List[int]
        """
        mp = defaultdict(int)
        for arr in [nums1, nums2, nums3]:
            for val in set(arr):
                mp[val] += 1
        ans = []
        for key, val in mp.items():
            if val >= 2:
                ans.append(key)
        return ans
```
Complexity: `O(n)` where `n` is the sum of the the length of `nums1`, `nums2`, and `nums3`.
## [5895. Minimum Operations to Make a Uni-Value Grid](https://leetcode-cn.com/contest/weekly-contest-262/problems/minimum-operations-to-make-a-uni-value-grid/)
The optimal way is to change all the values to be their median.
```
class Solution(object):
    def minOperations(self, grid, x):
        """
        :type grid: List[List[int]]
        :type x: int
        :rtype: int
        """
        m, n = len(grid), len(grid[0])
        v = []
        for i in range(m):
            for j in range(n):
                v.append(grid[i][j])
        v.sort()
        tar = v[m * n // 2]
        ans = 0
        for val in v:
            diff = abs(val - tar)
            if diff % x:
                return -1
            ans += diff // x
        return ans
```
Complexity: `O((m + n)*log(m + n))` where `m` is the number of rows in `grid` and `n` is the number of columns in `grid`.
## [5896. Stock Price Fluctuation](https://leetcode-cn.com/contest/weekly-contest-262/problems/stock-price-fluctuation/)
For the `update` function, we need dict to capture the mapping from each timestamp to its corresponding price.
For the `current` function, we can just create class variables to keep track of the most recent timestamp and price.
To fetch the maximum/minimum price, we need a sorted list which support the query/update operation in `O(log(n))`. 
We have to be careful when dealing with the <b>correcting</b> case because all class variables may need to be updated.
```
from sortedcontainers import SortedList

class StockPrice(object):

    def __init__(self):
        self.mp = defaultdict(int)
        self.sl = SortedList()
        self.cur_ts = -1
        self.cur_p = -1

    def update(self, ts, price):
        """
        :type timestamp: int
        :type price: int
        :rtype: None
        """
        if ts in self.mp:
            p = self.mp[ts]
            self.sl.remove((p, ts))
            del self.mp[ts]
        self.mp[ts] = price
        self.sl.add((price, ts))
        if ts >= self.cur_ts:
            self.cur_p = price
            self.cur_ts = ts

    def current(self):
        """
        :rtype: int
        """
        return self.cur_p

    def maximum(self):
        """
        :rtype: int
        """
        return self.sl[-1][0]

    def minimum(self):
        """
        :rtype: int
        """
        return self.sl[0][0]


# Your StockPrice object will be instantiated and called as such:
# obj = StockPrice()
# obj.update(timestamp,price)
# param_2 = obj.current()
# param_3 = obj.maximum()
# param_4 = obj.minimum()
```
Complexity: 
1. `init`, `current`: `O(1)`
2. `update`, `maximum`, and `minimum`: `O(log(n))` where `n` is the number of timestamps so far.
## [5897. Partition Array Into Two Arrays to Minimize Sum Difference](https://leetcode-cn.com/contest/weekly-contest-262/problems/partition-array-into-two-arrays-to-minimize-sum-difference/)
This is a really good question. Assume the arrays to create are `array1` and `array2`.

First, we need to adapt [meet in the middle](https://www.geeksforgeeks.org/meet-in-the-middle/) technique. (`1 <= n <= 15` and `nums.length == 2 * n` are the hits) 
We use the mask in length `n` to populate all possible split of `array1` and `array2` using the elements in the first half and the second half. 
For each position in the mask, `1` presents being choosed for the first array, otherwise, the second array.

Second, we need to consider `n` combinations. 
For example, for `array1`, we need to combine `x` elements from the first half with `y` elements from the second half, where `x + y == n`.
Therefore, we create `pre_sum` and `suf_sum` with `n` sets to store the difference between `array1` and `array2`:
1. `pre_sum`: the difference between `array1` and `array2` from the first half (e.g., index < `n`).
2. `suf_sum`: the difference between `array1` and `array2` from the second half (e.g., `n` <= index < `len(nums)`).
The set structure is used for reducing the cost of time and space.

Finally, when matching the first half and the second half for `array1` and `array2`, we use binary search to find the minimum difference. 
For each `val` in `pre_sum[x]`, we simply search `suf_sum[n - x]` for the smallest value that greater or equal to `-val`.
Considering that there are two possiblities: positive difference and negative difference, we should also consider the largest value that smaller than `-val`.
```
class Solution(object):
    def minimumDifference(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums) // 2
        pre_sum = [set() for _ in range(n + 1)]
        suf_sum = [set() for _ in range(n + 1)]
        for mask in range(1 << n):
            pre_val = 0
            suf_val = 0
            cnt = 0
            for i in range(n):
                if mask & (1 << i):
                    pre_val += nums[i]
                    suf_val += nums[i + n]
                    cnt += 1
                else:
                    pre_val -= nums[i]
                    suf_val -= nums[i + n]
            pre_sum[cnt].add(pre_val)
            suf_sum[cnt].add(suf_val)
        for i in range(n + 1):
            suf_sum[i] = list(suf_sum[i])
            suf_sum[i].sort()
        ans = 10**10
        for i in range(n):
            for val in pre_sum[i]:
                idx = bisect.bisect_left(suf_sum[n - i], -val)
                if idx < len(suf_sum[n - i]):
                    ans = min(ans, abs(val + suf_sum[n - i][idx]))
                if idx > 0:
                    ans = min(ans, abs(val + suf_sum[n - i][idx - 1]))
        return ans
```
Complexity: `O(n * (2^n) * log(2^n))` where `n` is half length of `nums`.
