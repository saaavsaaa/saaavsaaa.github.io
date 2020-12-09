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
但是如果只是这么做，极端情况下，树高就等于节点数，那么遍历所有节点的复杂度是1+2+...+n(因为叶子节点要旋转n次)，复杂度是O(n²)，均摊就是O(n)




-----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract2.md)
