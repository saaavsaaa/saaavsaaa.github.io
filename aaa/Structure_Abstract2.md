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


-----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract2.md)
