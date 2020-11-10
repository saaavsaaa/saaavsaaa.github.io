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

```

float evaluate( char* S, char* & RPN ) {         //中缀表达式求值       ,RPN是可以顺便生成的逆波兰表达式    
   Stack<float> opnd; Stack<char> optr;                  //运算数栈、运算符栈   
   optr . push('\0');                             //尾哨兵'\e' 也作为头哨兵首先入栈   
   while ( !optr . empty() ) {                 //逐个处理各字符，直至运算符栈空   
      if ( isdigit( *S ) )                     //若当前字符为操作数，则   
         readNumber( S, opnd );                 //读入 (可能多位的)操作数   
      else                            //若当前字符为运算符，则视其与栈顶运算符之间优先级的高低   
         switch( orderBetween( optr. top(), *S ) ) { /*分别处理*/| }   
   } //while   
   return opnd. pop();                         //弹出并返回最后的计算结果   
}   

const char pri[N_ _OPTR][N_ OPTR] = {}

```
```

switch( orderBetween( optr . top(), *S ) ) {   
   case '<':                                       //栈顶运算符优先级更低     
      optr.push( *S ); S++; break;                         //计算推迟,当前运算符进栈     
   case '=':                                        //优先级相等(当前运算符为右括号，或尾部哨兵'\0' )       
      optr .pop(); S++; break;                              //脱括号并接收下一 个字符   
   case '>': {                                     //栈顶运算符优先级更高,实施相应的计算，结果入栈   
      char op = optr .pop();                               //栈顶运算符出栈，执行对应的运算   
      if ( '!' == op ) opnd.push( calcu( op, opnd.pop() ) );                 //一元运算符   
      else { float pOpnd2 = opnd.pop()，pOpnd1 = opnd.pop();                 //二元运算符   
         opnd.push( calcu( pOpnd1， op， pOpnd2 ) );                  //实施计算，结果入栈   
      }            //为何不直接 :opnd .push( calcu( opnd.pop(), op, opnd.pop() ) )?  因为反了   
      break;   
   } //case '>'   
}//switch   

```

(1+2^3!-4)*(5!-(6-(7-(89-0!))))$ = 2013

#### RPN 逆波兰表达式( Reverse Polish Notation )     
J. Lukasiewicz ( 12/21/1878 - 02/13/1956 )   
在由运算符( operator )和操作数( operand )组成的表达式中   
不使用括号( parenthesis-free )，即可表示带优先级的运算关系   

转换为逆波兰表达式：   
1)添加括号，用括号显式地表示优先级   
   {([0!]+1)^([(2*[3!] ) +4] -5)}   
2)将运算符移到对应的右括号后   
   {([θ]! 1)+([ (2[ 3 ]!)* 4] + 5 )-} ^    
3)抹去所有括号   
   0! 1+ 2 3! * 4 + 5 - ^   

数字顺序不变

```

float evaluate( char* S, char* & RPN ) { //RPN转换   
   /* ......................................... */   
   while ( !optr . empty() ) { //逐个处理各字符，直至运算符栈空   
      if ( isdigit(*S) ) //若当前字符为操作数,则直接将其   
         { readNumber( S, opnd ); append(RPN, opnd.top() ); } //接入RPN   
      else //若当前字符为运算符   
         switch( orderBetween( optr . top(), *S ) ) {   
            /* ......................................... */   
            case '>': { //且可立即执行,则在执行相应计算的同时将其   
               char op = optr . pop(); append( RPN, op ); //接入RPN   
            /* .................................. */   
         } //case '>'   
         /* .................................. */   

```
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/index.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/rpn.h.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/append2rpn.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/calculation.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/displayprogress.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/main.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/priority.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/priority.h.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/readnumber.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/rpn.cpp.htm   
https://dsa.cs.tsinghua.edu.cn/~deng/ds/src_link/rpn/rpn.h.htm   

#### 队列
队列( queue )也是受限的序列   先进先出( FIFO) 后进后出( LILO )   
只能在队尾插入(查询) : enqueue() + rear()   
只能在队头删除(查询) : dequeue() + front()   
队列既然属于序列的特列,故亦可直接基于向量或列表派生     
```
template <typename T> class Queue: public List<T> { //由列表派生的队列模板类
public: //size()与empty()直接沿用
   void enqueue(T const & e) { insertAsLast(e); } //入队
   T dequeue() { return remove( first() ); } //出队
   T & front() { return first()->data; } //队首
}; //以列表首/末端为队列头/尾-------颠倒过来呢 ?
```
如此实现的队列接口，均只需O(1)时间   
用List颠倒过来无所谓
用向量头为队列头入队O(n)，出队O(1);尾为队列头入队O(1)，出队O(n)。（不考虑扩容的情况）   

C++ 课后题(由于审题错误，口算失败，直接写代码):
```
test.h:
#include <queue>

template<typename T> class XXX {
   public:
        void put(T const&);
        T remove();
   private:
        std::queue<T> Q1, Q2;
};

xxx.cpp:
#include "test.h"
#include <iostream>
using namespace std;

template<typename T> void XXX<T>::put(T const& e){ Q1.push(e);}

template<typename T> T XXX<T>::remove(){ //删除元素
                while( Q1.size() > 1 ){
                        Q2.push( Q1.front() );
                        Q1.pop();
                }
                T tmp = Q1.front();
                Q1.pop();
                while(!Q2.empty()){
                        Q1.push(Q2.front());
                        Q2.pop();
                }
                cout <<Q1.front();
                cout <<Q1.back();
                cout << tmp << endl;
                return tmp;
          }

int main()
{
                XXX<int> aaa;
                aaa.put(2);
                aaa.put(3);
                aaa.remove();
                aaa.put(5);
                aaa.put(7);
                aaa.remove();
                aaa.put(11);
                aaa.remove();
                aaa.remove();
}


```
g++ xxx.cpp -o aaa

### 二叉树
树结构可以理解为二维列表，半线性，与图的非线性有区别   
第一印象：以层次关系表示，如RPN 逆波兰表达式就可以用树表示，用运算符连接数字   
树是特殊的图T = (V, E)，节点数|V| = n,边数|E| = e   
指定任一节点r∈V作为根后, T即称作有根树( rooted tree )，与图论中讨论的树不同，这里都是有根树   
若: T<sub>1</sub>，T<sub>2</sub>, ...T<sub>d</sub>为有根树   
则:T= ( (∪V<sub>1</sub>)∪{r}, (∪E<sub>1</sub>)∪{ <r,r<sub>i</sub> | 1≤ i ≤d} )也是相对于T，T<sub>i</sub>称作以r<sub>i</sub>为根的子树( subtree
rooted at r<sub>i</sub>)，记作T<sub>i</sub> = subtree(r<sub>i</sub>)   
多颗树可以通过引入新根和新边组成一棵树    

#### 有序树
r 的子节点个数也就是连接子节点的边数 d = degree(r)为r的(出)度( degree )     
定义同级节点之间的顺序是计算机中的树和数学中研究的树的一个重要的不同之处   
可归纳证明 :(e) = ∑<sub>r∈V</sub>degree(r) = n-1 = Θ(n)
故在衡量相关复杂度时,可以n作为参照   
边数=节点数-1，用边和节点数的和度量树的规模，这个规模和边数、节点数在渐进意义下同阶   
所以之后讨论时间复杂度时都是以节点数为参照   
若指定T,作为T的第i棵子树，r<sub>i</sub>作为r的第i个孩子,则T称作有序树( ordered tree )，明确定义了同级节点间次序的   

#### 路径和环路
V中的k+1个节点,通过E中的k条边依次相联，构成一条路径( path 亦称通路), π={(V₀, v₁)， (V₁, V₂)，......，(Vₖ-₁, Vₖ) }     
路径长度:|π| = 边数 = k //早期文献，或以节点数为长度   
环路(cycle/loop) :Vₖ = V₀ ，短路    

节点之间均有路径,称作连通图( connected )   
不含环路，称作无环图( acyclic，acyclic graph )      
树:无环连通图，极小连通图、极大无环图，在连通和无环中达到一种平衡的图   
边越多越容易出现环路，无环边不能太多，连通边又不能太少   
故:任一节点v与根之间存在**唯一**路径path(v,r) = path(v)   
唯一指标：路径的长度   
指定了根节点后，同层节点的路径长度相同，每个节点的路径唯一   

不致歧义时,路径、节点和子树可相互指代。如：某个子节点有时可以代表以它为根的子树。   
path(v) ~ V ~ subtree(v)   
v的深度: depth(v) = | path(v) |，路径长度指标   
根节点到 v 的长度即 v 的深度，path(v)上节点，均为 v 的祖先( ancestor )，v 是它们的后代( descendent )   
其中，除自身以外,是真(proper)祖先/后代   
半线性:在任一深度，v 的祖先/后代若存在，则必然 / 末必唯一。线性前后都唯一；树前唯一，后不一定唯一；图前后都未必唯一。   
根节点是所有节点的公共祖先，深度为 0。没有后代的节点称作叶子（leaf)，所有叶子深度中的最大者的深度称作(子)树(根)的高度 height(v) = height( subtree(v) )，节点到叶子的长度叫高度   
特别地,空树的高度取作 -1。   
depth(v) + height(v) ≤ height(T)，在深度最深的叶子节点的路径上的点取等号   

#### 树的表示
|节点|功能|
|--:|:--|
|root()|根节点|
|parent()|父节点|
|firstChild()|长子|
|nextSibling()|兄弟|
|insert(i, e)|将e作为第i个孩子插入|
|remove(i)|删除第i个孩子(及其后代)|
|traverse()|遍历|

简单来说，先讲了用列表递归表示，就像数据表里用一张表表示多个层级的形式，下一级中有一个字段指向上一级的ID，但这种形式寻找子节点要遍历全集O(n)。进一步可以在包含子节点的节点上用指针指向所有子节点，美中不足是各个节点的子节点数量可能并不平均，子节点个数和子树根节点的出度同阶，只有平均了之后才能到O(1)，另外极端情况下可能子节点的规模达到n，比如所有节点都在根下。   
每个节点均设两个引用:   
纵: firstChild()   
横: nextSibling()   
使用根节点只管理长子节点，由长子节点管理其他同级节点，(每个节点管理的节点数平均又少了一些)   
其实极端情况也并没好多少，不过毕竟几乎不会那么极端。而且在后续的二叉树中很重要，以及二叉树在借助这种表示法在某些条件下可以表达所有树   

#### 二叉树
节点度数不超过2的树称作二叉树( binary tree )   
同一节点的孩子和子树,均以左右区分   
lChild() ~ 1Subtree()   
rChild() ~ rSubtree( )   
隐含有序   
深度为k的节点，至多2<sup>k</sup>个   
含n个节点、高度为h的二叉树中: h < n < 2<sup>h+1</sup>   
1)n = h+1时退化为一条单链   
2)n = 2<sup>h+1</sup> - 1时，即所谓满二叉树( full binary tree )宽度倍数增长相对快，高度对数增长慢   
所有节点的出度都是偶数，即子节点0或2，的二叉树叫真二叉树   

二叉树是多叉树的特例，但在有根且有序时,其描述能力却足以覆盖后者   
多叉树均可转化并表示为二叉树---回忆长子- 兄弟表示法   
之前说的用二叉树可以表示所有树的方法就是左子节点及左子节点的同级节点作为左子树，右子树是其同级节点链   

```
#define BinNodePosi(T) BinNode<T>y //节点位置
template <typename T> struct BinNode {
  BinNodePosi(T) parent, 1Child, rChild; //父亲、孩子
  T data; int height; int size(); //高度、子树规模
  BinNodePosi(T) insertAsLC( T const & ); //作为左孩子插入新节点
  BinNodePosi(T) insertAsRC( T const & ); //作为右孩子插入新节点
  BinNodePosi(T) succ(); // (中序遍历意义下)当前节点的直接后继1Ch
  template <typename VST> void trayLlevel( VST & ); //子树层次遍历
  template <typename VST> void travPre( VST & ); //子树先序遍历
  template <typename VST> void travIn( VST & ); //子树中序遍历
  template <typename VST> void travPost( VST & ); //子树后序遍历
```
```
template <typename T> BinNodePosi(T) BinNode<T>::insertAsLC(T const &e)
  { return lChild = newBinNode( e, this ); } // this.Lc == NULL
template <typename T> BinNodePosi(T) BinNode<T>::insertAsRC(T const & e)
  { return(rchild = new BinNode( e, this );}
template <typename T>
int BinNode<T>::size() { //后代总数,亦即以其为根的子树的规模
  int s = 1; //计入本身
  if (1Child) s += 1Child->size(); //递归计入左子树规模
  if (rChild) s += rChild->size(); //递归计入右子树规模
  return s;
} //0(n = |size|)
```
```
template <typename T> class BinTree {
protected:
  int _size; //规模
  BinNodePosi(T) _root; //根节点
  virtual int updateHeight( BinNodePosi(T) x ); //更新节点x的高度
  void updateHeightAbove( BinNodePosi(T) x ); //更新x及祖先的高度
public:
  int size() const { return _size; } //规模
  bool empty() const { return !_root; } //判空
  BinNodePosi(T) root() const { return _root; } //树根
  /* ...子树接入、删除和分离接口... */
  /* ...遍历接口... */
}
```
```
#define stature(p) ( (p) ? (p)->height :( -1) //节点高度一约定空树高度为
template <typename T> //更新节点x高度。具体规则因树不同而异
int BinTree<T>: :updateHeight( BinNodePosi(T) x) {
  return x -> height = 1 + max((stature x->1Child )，(stature( x->rChild ) );
} //此处采用常规二叉树规则，O(1)
template <typename T> //更新v及其历代祖先的高度
void BinTree<T>: :updateHeightAbove( BinNodePosi(T)() {
  while (x) //可优化:一旦高度未变,即可终止
    { updateHeight(x); x = x->parent; }
  } //0( n = depth(x) )
```
```
template <typename T> BinNodePosi(T)
BinTree<T>: :insertAsRC( BinNodePosi(T) x，T const &e) { //insertAsLC()对称
  _size++; x->insertAsRC(e); //x祖先的高度可能增加， 其余节点必然不变
  updateHeightAbove(x) ;
  return x->rChild;
}
```

#### 先序遍历
半线性结构   
通过遍历将非线性转为线性   
按照某种次序访问树中各节点每个节点被访问恰好一次
T=V ∪ L ∪ R
遍历结果~遍历过程~遍历次序~遍历策略 
先序 V L R   
中序 L V R   
后序 L R V   
就是从左到右，何时访问根，先根就是先序   
层次(广度) : 自上而下，先左后右 (后面会讲)   

递归实现
```
template <typename T， typename VST>
void traverse( BinNodePosi(T) x，VST & visit ) {
  if ( !x ) return;
  visit( x->data ) ;
  traverse( x->1Child, visit );
  traverse( x->rChild, visit );
} //T(n) = O(1) + T(a) + T(n-a-1) = O(n)
```
虽然渐进意义上与迭代差不多，但通用形式的栈帧常系数相差很大    
尾递归比较容易改成迭代      
迭代实现
```
template <typename T, typename VST>
void travPre_I1( BinNodePosi(T) x, VST & visit ) {
  Stack <BinNodePosi(T)> s; //辅助栈
  if (x) S.push(x); //根节点入栈
  while ( !S.empty() ) { //在栈变空之前反复循环
    x = S.pop(); visit( x->data ); //弹出并访问当前节点
    if ( HasRChild( *x ) ) S.push(. x->[rChild ); //右孩子先入后出
    if ( HasLChild( *x ) ) S.push( x->[1Child ); //左孩子后入先出
} //体会以上两句的次序 由于要先访问左，后进先出，所以左后进
```
这种方法不容易直接推广到中序和后序遍历     
只要访问当前节点，然后检查右节点，有右入栈，接着当前节点指向左子节点，一直遇到有右子节点就入栈，直到左子节点链到头，然后弹出栈重复此方式   
```
template <typename T, typename VST> //分摊O(1)
static void visitAlongLeftBranch(
  BinNodePosi(T) x,
  VST & visit,
  Stack <BinNodePosi(T)> & S )
  while (x) { //反复地
    visit( x->data ); //访问当前节点
    S.push( x->rChild ); //右孩子(右子树)入栈(将来逆序出栈)
    x = x->lChild; //沿左侧链下行
  } //只有右孩子、NULL可能入栈新的技个铁在登法除后者,是否值得?
}

template <typename T, typename VST>
void travPre_I2( BinNodePosi(T) x, VST & visit ) {
  Stack <BinNodePosi(T)> S; //辅助栈
  while(true) { //以(右)子树为单位,逐批访问节点
    visitAlongLeftBranch( x, visit, S ); //访问子树x的左侧链,右子树入栈缓冲
    if ( S.empty() ) break; //栈空即退出
    x = S.pop(); //弹出下一子树的根
} //#pop = #push = #visit = O(n) =分摊O(1)
```
###### 中序遍历
```
template <typename T，typename VST>
void traverse( BinNodePosi(T) x，VST & visit ) {
  if(!x)return;
  traverse( x->lChild, visit );
  visit( x->data );
  traverse( x->rChild, visit );
} //T(n) = T(a) + O(1) + T(n-a-1) = O(n) 
```
访问左子树不是尾递归，难以直接改迭代   
可以通过顺着左子节点链深入，不断入栈左子节点，直到到达最左端子节点，读取该节点，并入栈右子节点及其子树（访问子树方法同前），如无右子节点则弹栈并访问该节点，然后访问弹出节点右子树，以此类推   
```
template <typename T>
static void(goAlongLeftBranch( BinNodePosi(T) x, Stack <BinNodePosi(T)> & S )
{ while (x) {S.push(x);x = x->1Child; } } //反复地入栈,沿左分支深入

template <typename T, typename V> void travIn_I1( BinNodePosi(T) x，V& visit ) {
  Stack <BinNodePosi(T)> S; //辅助栈
  while (true) { //反复地
    goAlongLeftBranch( x, S ); //从当前节点出发,逐批入栈
    if ( S.empty() ) break; //直至所有节点处理完毕
    x = S.pop(); //x的左子树或为空，或已遍历(等效于空) ,故可以
    visit( x->data ); //立即访问之
  x = x->rChild; //再转向其右子树(可能为空，留意处理手法)
  }  
}
```
这是一个嵌套循环，循环次数与push执行次数相当，O(n)     

##### 层次遍历
从上至下，同层从左到右，由于是顺序的，所以使用与栈方向相反的结构--队列   

```
template <typename T> template <typename VST>
void BinNode<T>::travLevel( VST & visit ) { //二叉树层次遍历
  Queue<BinNodePosi(T)> Q; //引入辅助队列
  Q.enqueue( this ); //根节点入队
  while ( !Q.empty() ) { //在队列再次变空之前，反复迭代
    BinNodePosi(T) x = Q.dequeue(); //取出队首节点 ,并随即
    visit( x->data ); //访问之
    if ( HasLChild(*x) ) Q.enqueue( x->[1Child ); //左孩子入队
    if ( HasRChild(*x) ) Q.enqueue( x->rChild ); //右孩子入队
    }
}
```

#### 重构
重构：重新构建，依据遍历结果，重新构建出二叉树   
数学归纳法，递归，分而治之DAC   

对于任何二叉树树，有中序遍历结果，再有先序遍历或后序遍历结果之一就可以重构出原二叉树，因为可以根据先序（根节点在首位）或后序（根节点在结尾）确定根节点，并在中序中定位根节点以区分开左右子树，不断递归重复这个过程就可以重构出二叉树了   

通常情况下，只有先序和后序是无法重构的，例如左子树或右子树为空的情况，就无法通过根节点进行区分，但是在**真二叉树**的情况下可以，因为真二叉树，左右子树要么都为空，要么都存在，那存在的情况下，就可以通过先序确定左子树的根（在第二位，根节点之后），左子树的根在后序中正好在左子树的结尾，刚好可以分开左右子树；同样后序中，右子树的根刚好在倒数第二位（根节点之前），在先序中它在右子树的第一位，同样可以分开左右子树，于是真二叉树的先后序都有时可以重构，   

### 图
#### 概念
G = (V;E) 图   
vertex: n = |V|  顶点   
edge|arc: e = |E|   边   
邻接：图中通过一条边连接起来的两点的关系   
关联：点和连接它的边的关系   
树和链表都是图的一种特例   
课程中不讨论顶点与自身连边形成自环的情形   

若邻接顶点u和v的次序无所谓则(u, v)为无向边undirected edge   
所有边均无方向的图,即无向图undigraph   
反之，有向图digraph中均为有向边directed edge   
u、v分别称作边(u, v)的尾(tail)、 头( head)   
既有有向边又有无向边的：mixed graph     

所谓的路径也就是由一系列的顶点按照依次邻接的关系构成的一个序列   
如果在一条通路中不含重复的节点称之为simple path简单路径   
路径 π <V0,V1,...,Vk >   
长度 |π| = k
环/环路:V0 = Vk 头尾是同一个节点 / 重合   
有向无环图{ DAG ) directed acydlic graph 不包含环路的有向图   
Eulerian tour 欧拉环路：有向图中全部有向边组成一个环路，从一点出发回到此点经过所有有向边，且只经过一次   
Hamiltonian tour 哈密尔顿环路：有向图，从一点出发经过所有顶点，且只经过一次

图接口大致三类
操作顶点的、操作边的、操作之间关系的   
图可以用邻接矩阵和关联矩阵表示：   
邻接矩阵：用矩阵形式表示点之间的关系，有n个点，邻接矩阵就是个n阶方阵，如果第i个点和第j个点有无向或双向边，则 a<sub>ij</sub>和a<sub>ji</sub>处为1，如果是单向则其中一个为1，没有的为0，如果有权重，则为权重值，对角线上是自环，例如：
| 0(默认值) | v0 | v1 | ... | vi | ... | vn |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|v0 | | | | | | |
|... | | | | | | |
|vj | | | | 1 | | |
|... | | | | | | |
|vn | | | | | | |   
通过行列可以直接看出出度和入度，vi列代表入度都有谁指向vi，行代表出度vi指向谁，上面矩阵代表vj指向vi   
关联矩阵：用顶点和边表示，n个顶点e条边n×e，每一列都是一条边，每行代表一个顶点，可知，一列中恰好只有两个是 1，其余都是0   





----------------------------------------------------------------------------------------------------
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract1.md)
