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
|Object        |      Ljava/lang/Object:|
|int\[]         |      \[I|
|Object\[]\[]    |      \[\[Ljava/lang/Object:|

一个类类型的描述符是这个类的内部名,前面加上字符L,后面跟有一个分号。例如,String的类型描述符为Ljava/lang/String;。而一个数组类型的描述符是一个方括号后面跟有该数组元素类型的描述符。


方法描述符是一个类型描述符列表,它用一个字符串描述一个方法的参数类型和返回类型。方法描述符以左括号开头,然后是每个形参的类型描述符,然后是一个右括号,接下来是返回类型的类型描述符,如果该方法返回 void,则是 V(方法描述符中不包含方法的名字或参数名)。

|源文件中的方法声明 | 方法描述符|
|:-:|:-:|
|void m(int i, float f) | (IF)V |
|int m(Object o) | (Ljava/lang/Object;)I |
|int[] m(int i, String s) | (ILjava/lang/String;)\[I |
|Object m(int[] i) | (\[I)Ljava/lang/Object; |
