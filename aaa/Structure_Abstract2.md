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


-----
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract2.md)
