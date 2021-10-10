# Weekly Contest 251
Chinese site ranking: 30  
Global site ranking: 66
## [1945. Sum of Digits of String After Convert](https://leetcode-cn.com/contest/weekly-contest-251/problems/sum-of-digits-of-string-after-convert/)
We can just concate the converted digits together and then calculate the sum of digits.
```
class Solution(object):
    def getLucky(self, s, k):
        """
        :type s: str
        :type k: int
        :rtype: int
        """
        s = ''.join([str(ord(c) - ord('a') + 1) for c in s])
        ans = 0
        for _ in range(k):
            ans = sum([int(c) for c in s])
            s = str(ans)
        return ans
```
## [1946. Largest Number After Mutating Substring](https://leetcode-cn.com/contest/weekly-contest-251/problems/largest-number-after-mutating-substring/)
We solve the question with greedy algorithm. 
To generate the largest number, we need to consider from the most significant position to the least significant position. 
Considering that the goal is to mutate over a substring, we will start at the first char that is greater after mutation and stop mutation at first char that is smaller after mutation.
