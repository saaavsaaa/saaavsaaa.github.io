### 串
#### ADT (Abstract Data Type)抽象数据类型
字符串结构简单，主要讲它相关的算法，串匹配，查找，重点 indexof   
字符串定义：由来自某个字母表 ∑ 的字符所组成的有限序列：S = a₀ a₁ a₂ ... a<sub>n-1</sub> ∈  ∑*    
线性序列，不要求字符互异，每一元素是一个字符，可用Vector或List实现   
特征鲜明，通常，字符的种类不多，而串长 = n >> |∑|   
比如：
| 视作字符串 | : | 字符组成只有 |
|:-:|:-:|:--|
| 英文文章| |['A' - 'z'] ∪['a' - 'z'] ∪ { ' ', '.'，',' ，... } |
|C++程序| : |{ 95个可打印字符 } ∪ { LF，CR }* |
|天然蛋白质| : |{ 21种氨基酸 }* |
|DNA| : |{A, C, G, T}* |
|RNA| : |{A, C, G, U}* |
|二进制| : |{0,1}* |

相等: S\[0，n) = \[0，m) 长度相等 (n = m)，且对应的字符均相同( S[i] = T[i] )   
子串: substr(i，k) = s\[i，i + k)， 0 ≤ i < n, 0 ≤ k 亦即，从S[i]起的连续 k 个字符   
前缀: S.prefix(k) = S.substr(0，k) = s\[0，k)，0 ≤ k ≤ n 亦即，S中最靠前的 k 个字符
后缀: S.suffix(k) = S.substr(n - k, k) = s\[n - k, n)，0 ≤ k ≤ n 亦即，S中最靠后的 k 个字符   
( 联系: S.substr(i， k) = S.prefix(i + k).suffix(k) )   
空串: S[0, n-0 ] 也是任何串的子串、前缀、后缀   
任何串都是自身的子串、前缀、后缀   
长度严格小于原串的子串、前缀与后缀也称作真子串、真前缀与真后缀   
```
i k ：秩，T P ：其他串
length()
charAt(i)  
substr(i，k)
prefix(k)
suffix(k)
concat(T)   拼接字符串
equal(T)
indexOf(P)  索引接口，之后大量讨论它的高效实现
```
#### 串匹配
记 n = |T| (全部文本的长度) 和 m = |P| (待从全部文本中找出的目标长度，不可视为常数)，通常有 n >> m >> 2(常数)   //比如，100,000 >> 100 >> 2   
Pattern matching 的几个层次：   
detection : P是否出现?
location : 首次在哪里出现?    //本章主要讨论的问题   
counting : 共有几次出现?      // find /c "2013" students.txt   
enumeration : 各出现在哪里?   //find "2013" students.txt   



-----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract3.md)
