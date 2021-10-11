# Weekly Contest 251
Chinese site ranking: 30  
Global site ranking: 66
## [1945. Sum of Digits of String After Convert](https://leetcode-cn.com/contest/weekly-contest-251/problems/sum-of-digits-of-string-after-convert/)
We can just concate the converted digits together and then calculate the sum of digits by `k` times.
``` python
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
Complexity: `O(n * k)` where `n` is the length of the string `s` and `k` is the convertion times.
## [1946. Largest Number After Mutating Substring](https://leetcode-cn.com/contest/weekly-contest-251/problems/largest-number-after-mutating-substring/)
We solve the question with the greedy algorithm. 
To generate the largest number, we need to iterate from the most significant digit to the least significant digit. (From left to right in the given number)
Considering that the goal is to mutate over a substring, we will start from the first char that is greater after mutation and stop at first char that is smaller after mutation.
``` python
class Solution(object):
    def maximumNumber(self, num, change):
        """
        :type num: str
        :type change: List[int]
        :rtype: str
        """
        v = [c for c in num]
        changed = False
        for i in range(len(v)):
            idx = ord(v[i]) - ord('0')
            # print(idx, change[idx])
            if change[idx] > idx:
                v[i] = str(change[idx])
                changed = True
            elif change[idx] < idx:
                if changed:
                    break
        return ''.join(v)
```
Complexity: `O(n)` where `n` is the length of the given number.
## [1947. Maximum Compatibility Score Sum](https://leetcode-cn.com/contest/weekly-contest-251/problems/maximum-compatibility-score-sum/)
We can just solve the question using depth first search (DFS). 
To save the time of comparing the answer from each student and each mentor, we use an m-by-m matrix `match` to present the <b>compatibility score</b>. `match[i][j]` means the compatibility score of the i-th student and the j-th mentor. 
For DFS, we use a mask to mark which mentor has been selected and the recursion will go through all possibilities.
``` python
class Solution(object):
    def maxCompatibilitySum(self, students, mentors):
        """
        :type students: List[List[int]]
        :type mentors: List[List[int]]
        :rtype: int
        """
        m = len(students)
        n = len(students[0])
        match = [[0] * m for _ in range(m)] #[mentor][student]
        for i in range(m):
            for j in range(m):
                for k in range(n):
                    if mentors[i][k] == students[j][k]:
                        match[i][j] += 1
        self.ans = 0
        
        def dfs(cur, mask, tot):
            if cur == m:
                self.ans = max(self.ans, tot)
                return
            for i in range(m):
                if mask & (1 << i):
                    continue
                dfs(cur + 1, mask | (1 << i), tot + match[cur][i])
            
        dfs(0, 0, 0)
        return self.ans
```
Complexity: `O(m * m * n + m!)` where `m` is the number of students and `n` is the number of questions in the survey.
## [1948. Delete Duplicate Folders in System](https://leetcode-cn.com/contest/weekly-contest-251/problems/delete-duplicate-folders-in-system/)
This is a very good question! We need a [Trie](https://en.wikipedia.org/wiki/Trie) data structure for this question.

First, we create the Trie `tree` by inserting all the nodes with `paths`. In each node, the first element is the mapping from the sub path name to the corresponding child's node. The second element presents whether the node should be deleted as defined in the question. 

Second, we need to go through the graph and summarize the pattern of all subtrees. To capture the pattern, we simply use post-order traversal to generate the <b>pattern string</b>. The nodes will be distinguished from each other by parentheses. Since the same pattern could result in different strings, we have to sort the children by the node names. A dict `mp` will be used to map the pattern string with its corresponding parent node. Based on the mapping results, we update the second element for the nodes to be deleted.

Finally, we collect the result by depth first search (DFS). Those nodes to be deleted will be ignored during the traversal.
``` python
class Solution(object):
    def deleteDuplicateFolder(self, paths):
        """
        :type paths: List[List[str]]
        :rtype: List[List[str]]
        """
        # initialization
        tree = [{}, False]
        for path in paths:
            cur = tree[0]
            for node in path:
                if node not in cur:
                    cur[node] = [{}, False]
                cur = cur[node][0]
        
        # process
        mp = defaultdict(list)
        
        def dfs(cur, s):
            mask = ''
            for nei in sorted(cur[0]):
                mask += '(' + dfs(cur[0][nei], nei) + ')'
            if mask:
                mp[mask].append(cur)
            return mask + s 
        
        dfs(tree, '')
        for key, v in mp.items():
            # print(key, len(v))
            if len(v) == 1:
                continue
            for node in v:
                node[1] = True
        
        # collect results
        self.ans = []
        
        def get_paths(cur, v):
            if v:
                self.ans.append([e for e in v])
            for nei in cur[0]:
                if cur[0][nei][1]:
                    continue
                v.append(nei)
                get_paths(cur[0][nei], v)
                v.pop()
        
        get_paths(tree, [])
        return self.ans
```
Complexity: `O(n)` where `n` is the number of sub paths in the given array `path`.
