## 数据结构下
### 二叉搜索树
继承了二叉树的形式，同时也继承了向量的有序性   
有序性更集中体现在它的一个子集平衡二叉搜索树（BBST,balanced binary search tree）   
概述包括：定义、特点、规范   

二叉搜索树数据项之间,依照各自的关键码彼此区分 call-by-key ,需要关键码之间支持大小比较与、相等比对   
数据集合中的数据项统一地表示和实现为词条entry形式
```
template <typename K, typename V> struct Entry { //词条模板类
   K key; v value; //关键码、数值
   Entry( K k = K(), V v = V() ) : key(k)，value(v) {}; //默认构造函数
   Entry( Entry<K, V> const & e ) : key(e.key)， value(e.value) {}; //克隆
   //比较器、判等器(从此,不必严格区分词条及其对应的关键码)
   bool operator< ( Entry<K, V> const & e ) { return key < e.key; } //小于
   bool operator> ( Entry<K, V> const & e ) { return key > e.key; } //大于
   bool operator==( Entry<K, V> const & e ) { return key == e.key; } //等于
   bool operator!=( Entry<K, V> const & e ) { return key != e.key; } //不等
}
```
顺序性:任一节点均不小于左后代，不大于其右后代   
单调性: BST的中序遍历序列，必然单调非降   
|  |  |  | 4 |  |  |  |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| | |↙| |↘| | |  
| |2| | | |6| |
|↙| |↘| |↙| |↘| 
|1| |3| |5| |7|
|↓|↓|↓|↓|↓|↓|↓|
|1|2|3|4|5|6|7|

这一性质，也是BST的充要条件//对树高 做数学归纳...   
符合这一性质的二叉树必然是BST   
模板类：
```
template <typename T> class BST : public BinTree<T> { //由BinTree派生
public: //以virtual1修饰 ,以便派生类重写
   virtual BinNodePosi(T) & search( const T & ); //查找
   virtual BinNodePosi(T) insert( const T & ); //插入
   virtual bool remove( const T & ); //删除
protected:
   BinNodePosi(T)  _hot; //命中上一级节点
   BinNodePosi(T) connect34( //3 + 4重构
      BinNodePosi(T), BinNodePosi(T), BinNodePosi(T),
      BinNodePosi(T), BinNodePosi(T), BinNodePosi(T), BinNodePosi(T));
   BinNodePosi(T) rotateAt( BinNodePosi(T) ): //旋转调整
```
查找：   
由于已经有序，而查找是从根节点开始，未命中的情况下每次能淘汰掉一边的子树（由于左子树一定小于根、右子树一定大于根），查找次数累计不过树的高度，查找过程参考中序遍历的结果可以发现，实际上类似于二分查找，以上面树为例，要查5，首先比根4大，所以左子树不用找了，然后比较右子树的根6，直接进入右子树的左子树；如果找不到，可以命中哨兵，哨兵数据为NULL   
减而治之:从根节点出发，逐步地缩小查找范围,直到发现目标(成功) , 或查找范围缩小至空树(失败),对照中序遍历序列可见,整个过程可视作是在仿效有序向量的二分查找   
```
template <typename T> BinNodePosi(T) & BST<T>::search(const T & e) //演示
{ return searchIn(_root, e,_hot = NULL ); } //从根节点启动查找,_hot直系上级节点

static BinNodePosi(T) & searchIn( //典型的尾递归，可改为迭代版
   BinNodePosi(T) & v, //当前(子)树根
   const T & e, //目标关键码
   BinNodePosi(T) & hot) //记忆热点
{
   if ( !v I (e == v->data) ) return v; //足以确定失败、成功,或者
   hot = v; //先记下当前(非空)节点,然后再...
   return searchIn( ( ( e < v->data ) ? v->lChild : v->rChild ), e, hot);
} //运行时间正比于返回节点v的深度，不超过树高O(h)
```
返回的引用值:   
成功时,指向一个关键码为e且真实存在的节点   
失败时,指向最后一次试图转向的空节点NULL   
失败时,不妨假想地将此空节点,转换为一个数值为e的哨兵节点   
如此，依然满足BST的充要条件;而且更重要地...这种方法可以将成功和失败的语义统一起来，hot指向上一级，命中正常或哨兵节点   
查询最坏情况，在BST退化成有序链表，且最小值作为根的情况下O(n)     

插入:   
先借助search(e)确定插入位置及方向,再将新节点作为叶子插入   
若e尚不存在，则: \_hot为新节 点的父亲 , v = search(e)为 \_hot对新孩子的引用(哨兵)   
```
template <typename T> BinNodePosi(T) BST<T>::insert( const T &e ) {
   BinNodePosi(T) & x = search( e ); //查找目标(留意_hot的设置)
   if ( !x){//既禁止雷同元素，故仅在查找失败时才实施插入操作
      x = new BinNode<T>( e, _hot ); //在x处创建新节点,以_hot为父亲
      _size++; updateHeightAbove( x ); //更新全树规模,更新x及其历代祖先的高度
   }
   return x; //无论e是否存在于原树中，至此总有x->data = e
} //验证:对于首个节点插入之类的边界情况，均可正确处置

```
删除：
```
template <typename T> bool BST<T>::remove( const T & e ) {
   BinNodePosi(T) & x = search( e ); //定位目标节点
   if ( !x ) return false; //确认目标存在(此时_hot为x的父亲)
   removeAt( x，_ hot); //分两大类情况实施删除,更新全树规模
   _size--; //更新全树规模
   updateHeightAbove(_ hot ); //更新_ hot及其历代祖先的高度
   return true;
}/删除成功与否，由返回值指示
```
当待删除目标只有一个子树为空时，删除后，用子树节点顶替它就好了   
```
template <typename T> static BinNodePosi(T)
removeAt( BinNodePosi(T) & x， BinNodePosi(T) & hot ) {
   BinNodePosi(T) W = x; //实际被摘除的节点,初值同x
   BinNodePosi(T) succ = NULL; //实际被删除节点的接替者
   if ( ! HasLchild( *x ) ) succ = x = x- > rChild; //左子树为空
   else if ( ! HasR)Child( Px ) ) succ = x = x->lChild; //右子树为空
   else { /* ...左右子树并存的情况，略微复杂些... */ }
   hot = w->parent; //记录实际被删除节点的父亲
   if ( succ ) succ->parent = hot; //将被删除节点的接替者与hot相联
   release( w->data ); release( W ); //释放被摘除节点
   return succ; //返回接替者
} //此类情况仅需O(1)时间
```
左右子树都存在的情况下，找到右子树中最小的，也就是右子树最左端的节点，与待删节点交换，然后再删除   
```
template <typename T> static BinNodePosi(T)
removeAt( BinNodePosi(T) & x，BinNodePosi(T) & hot ) {
   /* ..... */
   else { //若x的左、右子树并存，则
      W = W->succ(); swap( x->data, w->data ); //令*x与其后继*w互换数据
      BinNodePosi(T) u = w->parent; //原问题即转化为，摘除非二度的节点w
      (_u == x ? u->rChild : u->1Chi1d ) = succ = W -> rChild;
   }
   /* ..... */
}
```
删除方法中不包含循环，只有查找最左端节点时不超过树高h次，复杂度不过树高O(h)   
##### 平衡与等价
BST最坏情况下主要方法的复杂度都线性正比与树高O(h)，而暂时我们没有手段能控制树高，最坏情况下，树高可能等于节点数   
随机生成 ： 根据不同节点的插入顺序得到的个数，可能得到的树个数 n！个，平均复杂度O(logn)   
随机组成 ： 根据树的不同组成形式得到的个数，卡特兰数个catalan(n)，因为n!中有可能有不同的插入顺序生成同样结构的树的可能，平均复杂度O(n的平方根)   
所以，平均情况下，比通常的O(logn)要大一些，是O(sqrt(n))   
节点数目固定时,兄弟子树高度越接近平衡 , 全树也将倾向于更低   
由n个节点组成的二叉树,高度不低于1ogn - 恰为logn时 ,称作理想平衡。但实际中，很少会碰到完全二叉树(Complete Binary Tree
)和满树(Full Binary Tree)   
理想平衡出现概率极低、维护成本过高,故须适当地放松标准   
退一步海阔天空:高度渐进地不超过O(1ogn)，即可称作适度平衡   
适度平衡的BST，称作平衡二叉搜索树( BBST)   
BST的中序遍历有歧义性，不同的BST可能中序遍历结果一样，这些一样的BST，在BBST中称为相互等价的BST   
等价的BST，两个特点：   
上下可变:联接关系不尽相同 ,承袭关系可能颠倒(垂直方向)   
左右不乱:中序遍历序列完全一致，全局单调非降(水平方向)   
旋转zig(v)以顶点v为中心顺时针旋转，等价的BST，旋转前和旋转后(zigged)拓扑不同，但中序遍历相同   
zag(v)以顶点v为中心逆时针旋转,zagged   
包括AVL、红黑树在内的BBST都定义了某种适度平衡准则，有时树会超出准则，但可以通过一系列等价变换，变回来   
变换需要遵守两条规则：  
1.等价变换要局限在局部常数规模内，涉及变换的点是常数规模，计算时间也可以控制在常数规模   
2.重新恢复为BST的过程中，经过的变换次数不要太多，至多不能超过O(logn)   
AVL的删除操作刚达到及格线O(logn)，但插入可达O(1)；而红黑树(R/B)这两种操作都达到了常数   

#### AVL
理想平衡出现概率极低、维护理想平衡的成本过高,故须适当地放松标，树高至多O(logn)     
不平衡之后恢复到适度平衡的过程叫重平衡rebalance   
BBST核心技巧两条：一、如何选择适度标准；二、重平衡的算法   
BST的平衡因子：balFac(v) = height( 1c(v) ) - height ( rc(v) ）     
AVL树的适度标准：（发明者：G. Adelson-Velsky& E. Landis 1962) :[ ∀ v, | balFac(v) | ≤ 1 ]   
证明 height(AVL) = O(1ogn) 与 证明 n = Ω ( 2 ^ height(AVL) ) 等价   
高度为h的AVL树，至少包含S(h) = fib(h + 3) - 1个节点，斐波那契   
递推关系式：S(h)=1+S(h-1)+S(h-2)   
S(h)规模下限,1树根节点，-1、-2左右子树   
数学推导过程： S(h)+1=[S(h-1)+1]+[S(h-2)+1]   
T(h) = S(h)+1   
原式 => T(h) = T(h-1) + T(h-2)   
考察与斐波那契的项对应，从边界情况开始，只有一个节点：n=1, h=0, T(h)=S(h)+1=2实际树高＋1, 斐波那契第三项   
n=2, h=1, T(h)=S(h)+1=3, 斐波那契第四项    
归纳可见，是斐波那契后移3项fib(h + 3)   
斐波那契大致是[n = Ω (Φⁿ)](https://github.com/saaavsaaa/saaavsaaa.github.io/blob/master/aaa/Structure_Abstract.md)呈Φ的指数形式增长,写成对数形式h = O(logn)构成高度的上界，这刚好符合BBST适度平衡的要求，于是证明AVL符合BBST的要求，达到了适度平衡的标准   
接口：
```
#define Balanced(x) \ //理想平衡
( stature( (x).1Child ) == stature( (x).rChild ) )
#define BalFac(x) \ //平衡因子
( stature( (x).1Child ) - stature( (x).rChild ) )
#define AvlBalanced(x) \ //AVL平衡条件
((-2<BalFac(x))8&(BalFac(x)<2))

template <typename T> class AVL : public BST<T> { //由BST派生
public: // BST::search( )等接口，可直接沿用
   BinNodePosi(T) insert( const T & ); //插入重写
   bool remove( const T & ); //删除重写
};
```
删除节点造成失衡会引起直接前驱的失衡，但删除引起的失衡只会是因为删除了某个节点后继中比较短的一条边的节点，而树高和子树高是由较长的边决定的，但重平衡不容易   
插入造成失衡会造成所有前驱的失衡，因为它失衡是由于加长了最长的边，重平衡容易   

#### 插入
由于插入一个节点（v），它的直系前驱（p）不会失衡，所以距离插入节点最近的失衡节点，不会低于直系前驱的前驱（g）,同时可有多个失衡节点,最低者g不低于x祖父      
当v是g的“左子节点的左子节点”或“右子节点的右子节点”，就是p和v同侧时，只需要**单旋**，g经单旋调整后复衡，子树高度复原;更高祖先也必平衡,全树复衡   
当v是g“左的右”或“右的左”时，zig-zag或zag-zig，需要**双旋**，同样g重平衡后，g的所有前驱都适度平衡了   
```
template <typename T> BinNodePosi(T) AVL<T>::insert( const T & e ) {
   BinNodePosi(T) & x = search( e ); if ( x ) return x; //若目标尚不存在
   X = new BinNode<T>( e, _hot );_ size++; BinNodePosi(T) xx = x; //则创建x
//以下,从x的父亲出发逐层向上,依次检查各代祖先g
   for ( BinNodePosi(T) g = x->parent; g; g = g->parent )
   if ( !Av1Balanced( *g ) ) { //一旦发现g失衡,则通过调整恢复平衡
      FromParentTo( *g ) = rotateAt( tallerChild( tallerChild( g ) ) );
      break; //g复衡后 ，局部子树高度必然复原;其祖先亦必如此,故调整结束
   } else //否则(在依然平衡的祖先处) ,只需简单地
      updateHeight( g ); //更新其高度(平衡性虽不变,高度却可能改变)
   return xx; //返回新节点:至多只需一次调整
}
```
#### 删除
单旋：删除一般是由于删除了较短分支上的节点，最近的失衡节点可能是被删节点的直接前驱，由于删除后的重平衡可能导致删除节点所在子树的高度-1，所以重平衡可能导致该子树更高层次上的节点失衡（高层节点中删除节点的这一侧子树整体高度-1，而这一侧本身就是比较短的一侧），失衡不断向上传播，极端情况下可能达到logn次（习题解析【7-17】）   
双旋：zig-zag(与zag-zig对称)，与前面类似   
```
template <typename T> bool AVL<T>::remove( const T & e) {
   BinNodePosi(T) & x = search( e ); if ( !x ) return false; //若目标的确存在
   removeAt( x，_hot );_size--; //则在按BST规则删除之后，_hot及祖先均有可能失衡
//以下，从_hot出发逐层向上，依次检查各代祖先g
   for ( BinNodePosi(T) g = _ hot; g; g = g->parent ) {
      if ( ! Av1Balanced( *g ) ) //一旦发现g失衡,则通过调整恢复平衡
         g = FromParentTo( *g ) = rotateAt( tallerChild( tallerChild( g ) );
      updateHeight( g ); //并更新其高度
   } //可能需做过Ω(1ogn)次调整;无论是否做过调整,全树高度均可能下降
   return true; //删除成功
}
```
#### 3+4重构
单旋和双旋，逻辑上理解很容易，但实际中真的按逻辑去实现比较啰嗦，而无论怎么旋转，最后想要达到的效果都是中序遍历不变的最平衡形式，所以完全可以把插入、删除这两个逻辑过程中涉及的三个节点和这三个节点的子树共4棵，直接重组成结果的样式   
设g(x)为最低的失衡节点,考察祖孙三代:g~p~v，按中序遍历次序,将其重命名为:a<b<c，它们总共拥有互不相交的四棵(可能为空的)子树，按中序遍历次序,将其重命名为:T₀<T₁<T₂<T₃   
```
template <typename T> BinNodePosi(T) BST<T> : :connect34(
   BinNodePosi(T) a，BinNodePosi(T) b，BinNodePosi(T) c,
   BinNodePosi(T)T0，BinNodePosi(T)T1，BinNodePosi(T)T2，BinNodePosi(T)T3)
{
   a->1child = T0; if (T0)T0->parent = a;
   a->rChild = T1; if (T1)T1->parent = a; updateHeight(a);
   c->1Child = T2; if (T2)T2->parent = c;
   c->rChild = T3; if (T3)T3->parent = c; updateHeight(c);
   b->1Child = a; a->parent = b;
   b->rChild = c; c->parent = b; updateHeight(b);
   return b;//该子树新的根节点
}
```
统一调整:实现
```
template<typename T> BinNodePosi(T) BST<T>:: rotateAt( BinNodePosi(T) V ) {
   BinNodePosi(T) p = v->parent, g = p->parent; 
   if ( IsLChild( *p ) ) //zig
      if ( IsLChild( *v ) ) { //zig-zig
         p->parent = g->parent; //向上联接
         return connect34( v, p, g, v->1Child, V->rChild, p->rChild, g->rChild );
      } else { //zig-zag
         v->parent = g->parent; //向上联接
         return connect34( p, v, g, p->1Chi1d, v->1Child, v->rChild, g->rChild );
      }
   else ( /*.. zag-zig & zag-zag ..*/ }
}
```
AVL综合评价：   
优点：无论查找、插入或删除，最坏情况下的复杂度均为O(logn) O(n)的存储空间    
缺点：借助高度或平衡因子（人为引入平衡因子概念，伸展树就不需要）,为此需改造元素结构,或额外封装   
      实测复杂度与理论值尚有差距   
         插入/删除后的旋转，成本不菲   
         删除操作后,最多需旋转Ω(logn)次(Knuth :实际使用中，通常每5次左右会遇到一次，发生概率21%左右)   
         若需频繁进行插入/删除操作，未免得不偿失   
      插入和删除的复杂度严重不对等，单次动态调整后，全树拓扑结构的变化量可能高达Ω(logn)，红黑树插入删除就可以都控制在常数   

### 高级搜索树
#### 伸展树
AVL树需要处处小心，时时维护   
局部性 Locality: 刚被访问过的数据,极有可能很快地再次被访问。这一现象在信息处理过程中屡见不鲜   
BST :刚刚被访问过的节点，极有可能很快地再次被访问。下一将要访问的节点,极有可能就在刚被访问过节点的附近   
连续的m次查找(m >> n = |BST| )，采用AVL共需O(mlogn)时间
利用局部性,能否更快? 仿效链表，链表使用局部性的方法是，每访问一个节点就将该节点放到链表头，因为链表头部比尾部访问快很多   
对于BST，就是每访问一个节点，就通过旋转使该节点逐层上升，直至到达根节点，每访问一个就将它旋转到根   
但是如果只是这么做，极端情况下，树高就等于节点数，那么遍历所有节点的复杂度是1+2+...+n(因为叶子节点要旋转n次)，复杂度是Ω(n²)，均摊就是Ω(n)
节点v一旦被访问，随即转移至树根，一步一步往上爬，自下而上，逐层单旋 zig( v->parent ) 或 zag( V- >parent ) 直到v最终被推送至根   
旋转次数呈周期性的算术级数演变:每一周期累计Ω(n2), 分摊Ω(n)，远差与logn，还需要一些技巧上的调整，伸展策略已经完备了      

#### 双层伸展
技巧就是，对于双层单侧的形式(zig-zig或zag-zag)，g~p~v 不先旋转直接前驱 p，而是先旋转 p 的前驱 g，这样旋转结果就是对称形式的，比如zig-zig就会变成zag-zag，只有三层看不出来，如果是5层zig-zig形式的，例a-b-c-d-e，a为根，e是叶子，访问e
|  |  |  |  |  |  |  |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:| 
|1||||||a|
|2|||||b||
|3||||c|||
|4|||d||||
|5||e|||||

先旋转c，然后d，这时a-b-e-d-c，e是个转折
|  |  |  |  |  |  |  |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:| 
|1||||a|||
|2|||b|||| 
|3||e|||||
|4|||d||||
|5||||c|||

此时，e前驱的前驱是a，于是旋转a
|  |  |  |  |  |  |  |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:| 
|1||||b|||
||||↙||↘|| 
|2||e||||a| 
||||↘||||
|3||||d|||
|4|||||c||

再旋转d，e就成为树根了
|  |  |  |  |  |  |  |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:| 
|1||e|||||
||||↘||||
|2||||b||| 
||||↙||↘||
|3||d||||a|
|4|||c||||

可以发现，树变矮了，而原本的方法，树高是不变得，而且，原树的高度越高，这种方法访问之后的高度变化会越大   
由于每两个节点会折叠一个到对称侧，对上例，从左到右，就是原左侧链只剩下了原来从底向上的第偶数个节点，奇数节点全都去了右侧，那么从倒数第3个节点开始，经过旋转每两层就会少一层。在不考虑倒数的3个节点的情况下，每经过一次从底到根，树高就会减少一半。   
折叠效果:一旦访问坏节点，对应路径的长度将随即减半   //含羞草：一旦遇到危险，遇到最坏情况就收缩起来   
最坏情况不致持续发生!   
单趟伸展操作，分摊O(logn)时间! /严格证明，详见习题[8-2]   
最后：   
在被访问节点 v 之上只有一层，无法双层伸展的情况下，此时必有parent(v) == root(T)，且每轮调整中，这种情况至多（在最后)出现一次视具体形态，做单次旋转:zig(r)或zag(r)，因此从渐近的意义而言并不会实质的影响整个调整过程的复杂度  
接口：   
```
template <typename T>
class splay : public BST<T> { //由BST派生
protected: BinNodePosi(T) splay( BinNodePosi(T) v );//将v伸展至根
public: //伸展树的查找也会引起整树的结构调整，故search()也需重写
   BinNodePosi(T) & search( const T & e );//查找重写
   BinNodePosi(T)insert(const T & e );//插入重写
   bool remove(const T & e );//删除重写
}
```
伸展算法：   
```
template <typename T> BinNodePosi(T) Splay<T>::splay( BinNodePosi(T) v ) {
   if ( ! v ) return NULL; BinNodePosi(T) p; BinNodePosi(T) g;//父亲、祖父
   while ( (p = v->parent) && (g = p->parent) ) { //自下而上，反复双层伸展
      BinNodePosi(T) gg = g->parent;//每轮之后，v都将以原曾祖父为父
      if ( IsLchild(* v ))
         if ( IsLChild(* p )){ /* zig-zig */ } else { /* zig-zag */ }
      else if ( IsRChild(* p) ){ /* zag-zag */ } else { /* zag-zig */ }
      if ( ! gg ) v->parent = NULL;//若无曾祖父gg，则v现即为树根;否则，gg此后应以v为左或右
      else ( g == gg->lc ) ? attachAsLChild(gg， v) : attachAsRChild(gg，v);//孩子
      updateHeight( g ); updateHeight( p ); updateHeight( v );
   }//双层伸展结束时，必有g == NULL，但p可能非空
   if ( p = v->parent ){/*若p果真是根，只需在额外单旋（至多一次)*/}
   v->parent = NULL: return v://迪展完成，v抵汰树相
}
```
四种情况(可参阅课后习题及课程附带的相关代码)：   
```
if(IsLChild(*v))
   if ( IsLChild( * p ) ) { //zIg-zIg
      attachAsLChild( g, p -> rc ); 
      attachAsLChild( p, v-> rc );
      attachAsRChild( p, g );
      attachAsRChild( v, p );
   } else { /* zIg-zAg */ }
else
   if(IsRChild(*p)){/* zAg-zAg */}
   else { /* zAg-zIg */ }
```
查找算法（如果未找到返回_hot，_hot应该就在根，后续的插入删除可以直接在根处操作）：   
```
template <typename T> BinNodePosi(T) & Splay<T>::search( const T & e )
//调用标准BST的内部接口定位目标节点
   BinNodePosi(T) p = searchIn(_ root, e, _hot = NULL );
//无论成功与否，最后被访问的节点都将伸展至根
   _root = splay( p ? p: _hot ); //成功、失败
//总是返回根节点
   return_ root;
}
```
插入：   
重写后的Splay::search()已集成了splay( )操作   
查找(失败)之后，_hot即是根节点   
具体的插入与普通的BST插入根节点一样   
删除：   
同样地, Splay: :search()查找(成功)之后，目标节点即是树根   
既如此, 何不随即就在树根附近完成目标节点的摘除...   
具体删除根可参考普通BST，比如找到右子树最小节点（右子树左侧链端点）   
综合评价：   
无需记录节点高度或平衡因子;编程实现简单易行一-优于AVL树   
分摊复杂度O(1ogn)--与AVL树相当   
局部性强、缓存命中率极高时(即k << n << m)，访问全局n个节点，一段时间内会只访问全局的一部分 k 个   
   效率甚至可以更高一自适应的O(1ogk)   
   任何连续的m次查找，都可在O(mlogk + nlogn) 时间内完成(aaa:操作次数远大于节点个数时，nlogn可以忽略)      
根据局部性原理，访问的数据会更多的在根节点附近命中，所以实际效率更高   
缺点：   
仍不能保证单次最坏情况的出现，因为树是不均衡的，最坏情况的调整代价很大   
不适用于对效率敏感的场合(任何一次的效率都要求很高的情况不适用),如手术室里器械相关的就不能合适   
复杂度的分析稍嫌复杂---这里只是简单分析，具体证明习题【8-2】   

### B树
严格来说B树并不是二分查找树，但逻辑上依然等效于BST，所以也归为高级搜索树，它可用于高效的I/O   
640K ought to be enough for anybody. - Bill. Gates，1981（在虚拟内存、内存交换和内存分页这三者结合下，运行一个程序，“必须”的内存很少。CPU只需要执行当前的指令，极限情况下，内存也只需要加载一页。其他的只在用到对应的数据和指令时，从硬盘交换到内存。以4k内存一页的大小，640K内存能放下足足160页，理论上足以运行任何规模的程序）   
RAM :存储器?不就是无限可数个寄存器吗?   
Turing:存储器?不就是无限长的纸带吗?   
实际中不可能有无限个，无限是一种过于理想的假设   
但事实上,系统存储容量的增长速度 << 应用问题规模的增长速度   
典型的 数据库规模/内存容量:   
   1980   :   10MB / 1MB = 10   
   2000   :   1TB / 1GB = 1000   
今天典型的数据集须以TB为单位度量:    
   345 TB ^ Global climate   
   300 TB ^ Nuclear   
   250 TB ^ Turbulent combustion   
   50 TB  ^ Parkinson's disease   
   10 TB  ^ Protein folding   
亦即，相对而言，内存容量是在不断减小!    
而内存不能做大点的原因是，容量越大，访问速度越慢，不得不在存储器的容量与它的访问速度之间做一取舍折中   
使用高速缓存解决容量和速度的矛盾，将存储分为多层，越向下层越大   
事实1 :不同容量的存储器,访问速度差异悬殊   
以磁盘与内存为例: ms量级(10<sup>-3</sup>)/ns量级(10<sup>-9</sup>) > 10<sup>5</sup>,保守估计至少也是5个数量级     
从前面学过的封底估算可知，若一次内存访问需要一秒，则一次外存访问就相当于一天，相当于从眼前那一只粉笔和坐火车跑半个中国买只粉笔回来   
为避免 1 次外存访问,我们宁愿访问内存10次、100次，甚至...   
多数存储系统，都是分级（CPU高速缓存、内存RAM、DISK磁盘、DISK ARRAY磁盘阵列等）组织的一一Caching，最常用的数据尽可能放在更高层、更小的存储器中，实在找不到，才向更低层、更大的存储器索取   
上下级存储之间访问速度差异巨大，下级相对于上级就是外存，需要尽量减少对外存的访问次数   
事实2 :从磁盘中读写1B，与读写 1KB 几乎一样快，就像坐火车采购一只和一盒差不多     
批量式访问:以页( page )或块( block )为单位,使用缓冲区//<stdio.h>...   
C 语言缓冲区接口，可以自定义缓冲区一次读取大小
```
#define BUFSIZ 512 //缓冲区默认容量
int setvbuf( //定制缓冲区
   FILE* fp, //流
   char* buf, //缓冲区
   int _Mode, // _IOFBF | _IOLBF | _IONBF 工作模式
   size_ t size); //缓冲区容量
int fflush(FILE* fp); //强制清空缓冲区
```
或者不读取，要读取就一次读取若干KB；坐火车要不就不买，买就买很多   

B树特点：1.每个节点不一定又几个分支；2.所有叶子节点高度一样（从这个角度看不失为一个理想平衡的搜索树）；3.相较于同样节点树的二叉树更宽、更矮；   
平衡的多路(multi -way)搜索树，实际与BST二路搜索树是等价的   
可以将B树节点看做一个超级节点     
经适当合并,得超级节点（每个超级节点含有多个关键码）   
   每2代合并: 4路    
   每3代合并:8路   
   每d代合并:m = 2<sup>d</sup>路，m - 1个关键码   
逻辑上与BBST完全等价，使用B树的原因主要是多级存储系统中使用B树，可针对外部查找,大大减少I/O次数   

对于AVL，若有n = 1G个记录...最坏情况下每次查找需要 1og(2, 10<sup>9</sup>) = 30 次I/O操作，每次只读出一个关键码,得不偿失   
B树充分利用外存对批量访问的高效支持，将此特点转化为优点，每下降一层,都以超级节点为单位，读入一组关键码

具体一组多大视磁盘的数据块大小而定（超级节点大小取决于磁盘等外存本身所设定的数据缓冲页面的大小，数据缓冲页一般若干KB，如果每个关键码4个字节，那很超级节点自然取200 ~ 300），m = #keys / pg。比如，目前多数数据库系统采用m=200 ~ 300    
回到上例，若有n = 1G个记录，若取m= 256，则最坏情况，查找只需 log(256, 10^9) <= 4 次I/O
虽然4和30都是常数，渐近意义下差不多，但常数的单位到达10的5、6次方时，就需要考虑了，比如一年和一秒都时常数   

定义m 阶B树，即 m 路 平衡搜索树(m ≥ 2)   
   外部节点的深度统一相等（哨兵节点，不包含关键码，B树和其它普通树不同，计算树的高度时需要把外部节点考虑进去）   
   所有叶节点的深度统一相等   

阶数给除了上限和下限，内部节点各有：   
   不超过 m-1 个关键码:  K₁ < K₂ <...< Kn   
   不超过 m 个分支: A₀, A₁, A₂,..., An   
下限：内部节点的分支数n + 1也不能太少，具体地    
   树根:2 ≤ n+1   
   其余:m/2 ≤ n+ 1   

既然如此我们也用超级节点所拥有分支数的下限、上限来命名 B 树：(m/2, m)树，（向上取整）   
   如5阶(也就是最多5个分支，4个关键码):(3,5)树；6阶:(3,6)树；7阶:(4,7)树...    
   其中(2,4)树与红黑树关系密切   

BTNode:   
一个超级节点可以用两个向量实现，一个存放n个关键码，另一个存放n+1个分支引用   
```
template <typename T> struct BTNode { //B-树节点
  BTNodePosi(T) parent; //父
  Vector<T> key; //数值向量
  Vector< BTNodePosi(T) > child; //孩子向量(其长度总比key多一 )
  BTNode() { parent = NULL; child. insert( 0, NULL ); }
  BTNode( T e, BTNodePosi(T) 1c = NULL, BTNodePosi(T) rc = NULL )
    parent = NULL; //作为根节点，而且初始时
    key.insert( 0, e ); //仅一个关键码，以及
    child.insert( 0, lc ); child.insert( 1, rc ); //两个孩子
    if ( 1c ) lc->parent = this; if ( rc ) rc->parent = this ;
}
```
BTree:   
```
#define BTNodePosi(T) BTNode<T>* //B-树节点位置
template <typename T> class BTree { //B-树
protected :
  int _size; int _order; BTNodePosi(T) _root; //关键码总数、 阶次、根
  BTNodePosi(T) _hot; //search( )最后访问的非空节点位置
  void solveOverflow( BTNodePosi(T) ); //因插入而上溢后的分裂处理
  void solveUnderflow( BTNodePosi(T) ); //因删除而下溢后的合井处理
public:
  BTNodePosi(T) search(const T & e); //查找:
  bool insert(const T & e); //插入
  bool remove(const T & e) //删除
}
```
查找(逐层深入，减而治之)：   
外部节点可能不存在，也可能是更低一层的外部存储器   
查找过程就是IO操作和向量查找交替的过程   
如果查找失败，必然失败于外部节点   
```
template <typename T> BTNodePosi(T) BTree<T>::search( const T & e) {
  BTNodePosi(T) v = _ root; _hot = NULL; //从根节点出发
  while(v) { //逐层查找
    Rank r = v->key.search(e); //在当前节点对应的向量中顺序查找
    if ( 0 <= r && e == v->key[r]) return v; //若成功，则返回;否则...
    _hot = v; v = v->child[r + 1]; //沿引用转至对应的下层子树，并载入其根I/O
  } //若因!v而退出，则意味着抵达外部节点
  return NULL; //失败
}
```
向量的查找用的是顺序查找，在此场景下，由于内外存访问的巨大差异，二分查找的未必更好，甚至可能更差，一个超级节点规模是几百，试验表明，这个规模下顺序可能比二分更好   
##### B树可能的最大高度：   
树根为第0层，外部节点第h层   
含 N 个关键码的 m 阶B树，为了让树高度最大，内部节点应尽可能“瘦”,各层节点数依次为   
  n₀=1，n₁=2, n₂=2x(m/2), ... ,nₖ = 2 x (m/2)<sup>k-1</sup>   
N个内部节点，N+1个外部节点   
N种成功可能，N+1种失败可能   
考查外部节点所在层   
  N+1 = nₕ ≥ 2 x (m/2)<sup>h-1</sup>  (nₕ的确界和下界) 
  h ≤ 1 + 1og<sub>m/2</sub>(N + 1)/2 = O(logₘN)
1ogₘN中的底数可以视为常数，与BST的性能渐近同阶，B树的意义并不在于降低搜索的渐近时间复杂度，而是更加关注于常系数意义下的优化   
相对于BBST :   
  log<sub>m/2</sub>(N/2) / 1og₂N = 1/(1og₂m - 1)   
若取 m = 256 树高 (I/O次数) 约降低至1/7    
##### B树高度的下界：   
类似的   
含 N 个关键码的 m 阶B树，为了让树高度最小，内部节点应尽可能"胖", 分支数不得超过m   
各层节点数依次为:   
  n₀=1, n₁=m, n₂=m², n₃ = m³, ... , n<sub>h-1</sub>=m<sup>h-1</sup> , nₕ = m<sup>h</sup>   
考查外部节点]所在层:   
  N+1 = nₕ ≤ m<sup>h</sup>   
  h ≥ logₘ(N + 1) = Ω(logₘN)   
相对于BBST: (logₘN - 1) / log₂N = logₘ2 - log<sub>N</sub>2 ≈ 1/log₂m     
若取 m = 256 树高 (I/O次数) 约降低至1/8     

综上，关键码数量固定时，B树高度的上下浮动范围是非常有限的，几乎可以忽略树高的变化   
##### 插入   
```
template <typename T>
  bool BTree<T>::insert( const T & e ) {
  BTNodePosi(T) v = search(e); 
  if(V) return false; //确认e不存在
  Rank r = _hot->key.search( e ); //在节点_hot中确定插入位置
  _hot->key.insert( r + 1, e]); //将新关键码插至对应的位置
  _hot->child.insert( r + 2, NULL ); //创建一个空子树指针
  _size++; solveOverflow( hot ); //如发生上溢，需做分裂
  return true; //插入成功
}
```
可以约定在_hot(key.search向量查找方法，返回不大于e的最大关键码)右侧引用插入   
另外，r+2(可以约定给新关键码增加右侧分支引用[指向下级节点的])处插入空节点的操作也可以用直接在向量尾部加，因为e不存在，那么一定命中了外部节点   
插入节点可能导致上溢出(超出B树阶次的规定)，需要进行分裂   

发生了上溢的超级节点应该刚好有m个关键码、m+1个分支(多了一个)，设上溢节点中的关键码依次为：k₀, k₁, k₂,..., k<sub>m-1</sub>   
取中位数s= m/」, 以关键码 kₛ 为界划分为 【k₀, k₁, k₂,..., k<sub>s-1</sub> 】【 kₛ 】【 k<sub>s+1</sub>,..., k<sub>m-1</sub> 】   
关键码 kₛ 上升一层,并分裂spiit:以所得的两个节点作为左、右孩子   





-----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract2.md)
