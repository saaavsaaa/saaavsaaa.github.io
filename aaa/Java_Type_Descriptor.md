类型描述符:

|Java 类型     |   类型描述符|
|:-:|:-:|
|boolean       |       Z|
|char          |      C|
|byte          |      B|
|short         |      S|
|int           |      I|
|float         |      F|
|long          |      J|
|double        |      D|
|Object        |      Ljava/lang/Object|
|int\[]         |      \[I|
|Object\[]\[]    |      \[\[Ljava/lang/Object|

一个类类型的描述符是这个类的内部名,前面加上字符L,后面跟有一个分号。例如,String的类型描述符为Ljava/lang/String;。而一个数组类型的描述符是一个方括号后面跟有该数组元素类型的描述符。
