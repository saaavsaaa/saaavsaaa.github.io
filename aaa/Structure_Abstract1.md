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

##### 栈的应用
逆序输出   conversion     输出次序与处理过程颠倒;递归深度和输出长度不易预知     
递归嵌套   stack permutation + parenthesis         具有自相似性的问题可递归描述，但分支位置和嵌套深度不固定    
延迟缓冲   evaluation         线性扫描算法模式中，在预读足够长之后，方能确定可处理的前缀     
栈式计算   RPN     基于栈结构的特定计     




----------------------------------------------------------------------------------------------------
[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Structure_Abstract1.md)
