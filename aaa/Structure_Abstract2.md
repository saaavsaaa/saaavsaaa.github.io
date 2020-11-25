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




-----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract2.md)
