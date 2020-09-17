## 栈与队列
都是线性序列的特例     
栈只能操作顶部元素     
汉诺塔的每个柱子都是一个有顺序要求的栈     

栈既然属于序列的特例,故可直接基于向量或列表派生     
template <typename T> class Stack public VectorkT> { //由向量派生的模板类
public: //size()、 empty( )以及其它开放接口均可直接沿用     
      void push(T const & e) { insert(size(), e); } //入栈     
      T pop() { return remove(size() - 1); } //出栈     
      T & top() { return (*this)[size() - 1]; } //取顶      
}; //以向量首/末端为栈底/顶 --------- 颠倒过来呢?     
向量需要以尾端为栈顶，链表没有限制     

#### 栈的应用
逆序输出   conversion     输出次序与处理过程颠倒;递归深度和输出长度不易预知     
递归嵌套   stack permutation + parenthesis         具有自相似性的问题可递归描述，但分支位置和嵌套深度不固定    
延迟缓冲   evaluation         线性扫描算法模式中，在预读足够长之后，方能确定可处理的前缀     
栈式计算   RPN     基于栈结构的特定计     

算法实现   
void convert( Stack\<char\> &S, \_int64 n, int base){   
      //新进制下的数位符号，可视base取值范围适当扩充 一>{ '0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F' };     
      static char digit[])=    
      while (n > 0) { //由低到高，逐一计算出新进制下的各数位       
            S.push( digit[n % base] ); //余数(对应的数位)入栈       
            n /= base; //n更新为其对base的除商     
}   

main() {   
      Stack\<char\> S;    
      convert(S, n, base); //用栈记录转换得到的各数位      
      while ( !S.empty() ) printf( "%c"， S.pop() ); //逆序输出   
}   

#### 括号匹配
逆序输出 conversion 输出次序与处理过程颠倒;递归深度和输出长度不易预知   
递归嵌套 stack permutation + parenthesis 具有自相似性的问题可递归描述，但分支位置和嵌套深度不固定   
延迟缓冲 evaluation 线性扫描算法模式中，在预读足够长之后，方能确定可处理的前缀   
栈式计算 RPN 基于栈结构的特定计算模式   

平凡情况，无括号，符合匹配   
对例子 : (()())()   
减而治之 如果先去了最前和最后的括号，就会出错   
分而治之 如果从中间位置分开，同样会出错   

颠倒以上思路:消去一对紧邻的左右括号,不影响全局的匹配判断   
那么，如何找到这对括号?   
再者，如何使问题的这种简化得以持续进行?   
顺序扫描表达式，用栈记录已扫描的部分  //实际上只需记录左括号   
反复迭代: 凡遇"("则进栈;凡遇")" ，则出栈   

以上思路及算法，可便捷地推广至多种括号并存的情况     
可否使用多个计数器?不行，反例:[(])   
甚至，只需约定“括号"的通用格式,而不必事先固定括号的类型与数目   
比如: \<body\>|\</body\>, \<h1\>|\</h1\>， \<font\>|\</font\>, \<p\>|\</p\>， \<ol\>|\</ol\>    

#### 栈混洗
卡特兰数 (2n)!/[(n+1)!×n!]     h(n)=C(2n,n)/(n+1):C(2n,n)=2n!/[n!(2n-n)!]     

栈 A =< a1, a2, ...an]、B=S=0  (左端为栈顶)    只允许:将A的顶元素弹出并压入s //s.push(A.pop()),或将S的顶元素弹出并压入B //B.push(S.pop())   若经过一系列以上操作后, A中元素全部转入B中 B = \[ ak1, .... akn >       (右端为栈顶)   则称之为A的一个栈混洗( stack permutation )   
同一输入序列，可有多种栈混洗，长度为n的序列，可能的混洗总数SP(n) = ？(参考组合数学卡特兰数 ) //显然，SP(n) <= n!     

输入序列<1，2,3,....n]的任-排列\[ P1, P2, P3, .... Pn >是否为栈混洗?   
简单情况:< 1，2，3],n= 3栈混洗共6!/4!/3!=5种   
全排列共3! =6 种   
少了一种\[ 3，1，2>   
//为什么是它?    
观察:任意三个元素能否按某相对次序出现于混洗中，与其它元素无关//故可推而广之 ! ...    
对于任何1 ≤ i < j < k ≤ n,\[ k...i...j >必非栈混洗   

充要性:A permutation is a stack permutation iff( Knuth, 1968 )it does NOT involve the permutation 312 //习题[4-3]   

如此，可得一个 O(n³) 的甄别算法) 进一步地...\[P1,P2,P3,..Pn> 是 <1,2,3，...n] 的栈混洗(当且仅当)对于任意i < j)不含模式 \[ ..., j + 1,..., i,..., j, ... >   
如此, 可得一个 O(n²)的甄别算法，再进一步地...   
O(n)算法:直接借助栈A、B和S，模拟混洗过程   
每次S.pop()之前，检测S是否已空:或需弹出的元素在S中，却非顶元素   

观察:每一栈混洗,都对应于栈S的 n次push 与 n次pop 操作构成的序列，将 push 和 pop 看做一对括号，一一对应，栈混洗就相当于括号匹配，n个元素的栈混洗有多少种，n对括号的**合法表达式**就有多少种     

#### 中缀表达式
逆序输出 conversion 输出次序与处理过程颠倒;递归深度和输出长度不易预知    
递归嵌套 stack permutation + parenthesis 具有自相似性的问题可递归描述，但分支位置和嵌套深度不固定   
延迟缓冲 evaluation 线性扫描算法模式中，在预读足够长之后，方能确定可处理的前缀   
栈式计算 RPN 基于栈结构的特定计算模式   

float evaluate( char* S, char* & RPN ) { //中缀表达式求值
Stack<float> opnd; Stack<char> optr; //运算数栈、运算符栈
optr . push('\0'); //尾哨兵'\e' 也作为头哨兵首先入栈
while ( !optr . empty() ) { //逐个处理各字符，直至运算符栈空
if ( isdigit( *S ) ) //若当前字符为操作数，则
readNumber( S, opnd ); //读入 (可能多位的)操作数
else //若当前字符为运算符，则视其与栈顶运算符之间优先级的高低
switch( orderBetween( optr. top(), *S ) ) { /*分别处理*/| }
} //while
return opnd. pop(); //弹出并返回最后的计算结果

const char pri[N_ _OPTR][N_ OPTR] =

switch( orderBetween( optr . top(), *S ) ) {
   case '<':                               //栈顶运算符优先级更低   
      optr.push( *S ); S++; break;          //计算推迟,当前运算符进栈   
   case '=':                                //优先级相等(当前运算符为右括号，或尾部哨兵'\0' )    
      optr .pop(); S++; break;           //脱括号并接收下一 个字符
   case '>': {                                 //栈顶运算符优先级更高,实施相应的计算，结果入栈
      char op = optr .pop();            //栈顶运算符出栈，执行对应的运算
      if ( '!' == op ) opnd.push( calcu( op, opnd.pop() ) );            //一元运算符
      else { float pOpnd2 = opnd.pop()，pOpnd1 = opnd.pop();            //二元运算符
         opnd.push( calcu( pOpnd1， op， pOpnd2 ) );            //实施计算，结果入栈
      }                                                        //为何不直接 :opnd .push( calcu( opnd.pop(), op, opnd.pop() ) )?  应该是因为反了
      break;
   } //case '>'
}//switch

(1+2^3!-4)*(5!-(6-(7-(89-0!))))$ = 2013


----------------------------------------------------------------------------------------------------
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract1.md)
