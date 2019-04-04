# KMP
> 字符串匹配算法真的很头疼，之前看KMP看了好多次，原理很简单，但是细节都没看明白，然后今天看了一个博客，才弄明白细节。

1. [KMP - LeetCode #459 Repeated Substring Pattern](https://www.cnblogs.com/liziran/p/6129336.html)
2. [从头到尾彻底理解KMP](https://blog.csdn.net/v_july_v/article/details/7041827)
## 算法原理
```
for(int i = 0; i < str1.size(); i++)
  for(int j = 0; j < str2.size();j++){
      if(str1[i] == str2[j]) i+=1;
      else{
              i = i - j;
              break;
      }
  }
```
我们在暴力匹配的时候，并没有充分利用到已遍历的子串的信息，当不匹配的时候，i回到i-j+1, j重新开始，如果匹配的子串具有两端对称性，我们遇到不匹配时，就不需要从0开始了。减少重复比较。例如 abcabd，如果我们匹配到abcab，我们发现下一个匹配不上，这个时候我们的j可以变成2，而不是0。因为我们知道在我们已经匹配到的子串里ab是最长前缀也是最长后缀，所以可以不再去匹配他们两个了。

那么我们怎么知道不匹配时，j应该变成多少呢，这就应该进行一个预处理。获得位置j不匹配时，他前面已匹配之串的前缀后缀长度。所以问题就转化成这个预处理了。

用数组next存储预处理的信息，对于一个匹配的子串，如果他们的前缀后缀长度是k，也就说S<sub>0</sub>S<sub>...</sub>S<sub>k-1</sub> = S<sub>j-k-1</sub>S<sub>...</sub>S<sub>j-1</sub> ,如果 S<sub>k</sub> = S<sub>j</sub>。那么next[j] = k + 1, 如果S<sub>k</sub> != S<sub>j</sub>，那么只能把范围缩的更小，也就是k'， 而这个k'只能是S<sub>0</sub>S<sub>...</sub>S<sub>k-1</sub>的最长前缀后缀，因为S<sub>0</sub>S<sub>...</sub>S<sub>k-1</sub> S<sub>...</sub><sub>j-k-1</sub>S<sub>...</sub>S<sub>j-1</sub> 是对称的。

如图
![](https://images2015.cnblogs.com/blog/1070510/201612/1070510-20161203182101459-93859599.png)

如果存在更小的k'，那么有
S<sub>0</sub>S<sub>...</sub>S<sub>k‘-1</sub>= <sub>j-k'-1</sub>S<sub>...</sub>S<sub>j-1</sub>。 又因为 S<sub>0</sub>S<sub>...</sub>S<sub>k-1</sub>=<sub>j-k-1</sub>S<sub>...</sub>S<sub>j-1</sub>。所以新的前缀必定是S<sub>0</sub>S<sub>...</sub>S<sub>k-1</sub> 的后缀，由此可证k’是S<sub>0</sub>S<sub>...</sub>S<sub>k-1</sub>的最长前缀后缀，即next[k]

```
class Solution:
    def get_next(self, s):
        "next[i] 储存的是除去当前字符的子串中，最长前缀后缀"
        next = [-1]
        j = 0
        k = -1
        while j < len(s) - 1:
            if k == -1 or s[k] == s[j]:
                k += 1
                j += 1
                next.append(k)
            else:
                k = next[k]
        return next
    
    def strStr(self, haystack: str, needle: str) -> int:
        next = self.get_next(needle)
        #print(next)
        #return
        i = 0
        j = 0
        while i < len(haystack) and j < len(needle):
            if(haystack[i] == needle[j]):
                i += 1
                j += 1
            elif j == 0: # 直接重新开始
                i += 1
            else:
                j = next[j]
                if j == -1:
                    j = 0
                    i = i + 1
        if j >= len(needle):
            return i - j
        else:
            return -1
```
