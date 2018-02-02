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

一个类类型的描述符是这个类的内部名,前面加上字符L,后面跟有一个分号。例如,String的类型描述符为Ljava/lang/String;(注意：有分号)。而一个数组类型的描述符是一个方括号后面跟有该数组元素类型的描述符。

-----

方法描述符是一个类型描述符列表,它用一个字符串描述一个方法的参数类型和返回类型。方法描述符以左括号开头,然后是每个形参的类型描述符,然后是一个右括号,接下来是返回类型的类型描述符,如果该方法返回 void,则是 V(方法描述符中不包含方法的名字或参数名)。

|源文件中的方法声明 | 方法描述符|
|:-:|:-:|
|void m(int i, float f) | (IF)V |
|int m(Object o) | (Ljava/lang/Object;)I |
|int[] m(int i, String s) | (ILjava/lang/String;)\[I |
|Object m(int[] i) | (\[I)Ljava/lang/Object; |

-----

    visit方法经常需要名称和描述符作为参数，我下面列的测试代码里有例子。Type.getType(***).getInternalName()可得到内部名，例：Type.getType(String.class).getInternalName() --> "java/lang/String"(此方法只能对类或接口类型使用)。Type.getType(***).getDescriptor()可获取一个Type的描述符，例：Type.getType(String.class).getDescriptor() --> "Ljava/lang/String;"。
    
    Type对象还可以表示方法类型。可以从方法描述符构建,也可以由Method对象构建。getDescriptor方法返回与这一类型对应的方法描述符。getArgumentTypes和getReturnType方法可用于获取与一个方法的参数类型和返回类型相对应的Type对象。例如,Type.getArgumentTypes("(I)V")返回一个仅有一个元素Type.INT_TYPE的数组。与此类似,调用Type.getReturnType("(I)V")将返回Type.VOID_TYPE对象。

-----

ClassVisitor 类的方法必须按以下顺序调用必须**首先**调用 visit,**然后**是对 visitSource 的**最多一个**调用,**接下来**是对visitOuterClass的**最多一个**调用,**然后**是可按**任意顺序**对 visitAnnotation 和visitAttribute 的任意多个访问,**接下来**是可按**任意顺序**对 visitInnerClass、visitField和visitMethod的**任意多个**调用,**最后以一个** visitEnd 调用结束。

-----

ASM 提供了三个基于 ClassVisitor API 的核心组件,用于生成和变化类:

    ClassReader 类分析以字节数组形式给出的已编译类,并针对在其 accept 方法参数中传送的 ClassVisitor 实例,调用相应的 visitXxx 方法。这个类可以看作一个事件产生器。

    ClassWriter 类是 ClassVisitor 抽象类的一个子类,它直接以二进制形式生成编译后的类。它会生成一个字节数组形式的输出,其中包含了已编译类,可以用toByteArray 方法来提取。这个类可以看作一个事件使用器。

    ClassVisitor 类将它收到的所有方法调用都委托给另一个 ClassVisitor 类。这个类可以看作一个事件筛选器。

-----


ClassReader用于分析已存在的类:  
https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassReaderTest.java

注意,构建ClassReader实例的方式有多种。必须读取的类可以用名字指定,也可以像字母数组或InputStream一样用值来指定。利用ClassLoader的getResourceAsStream方法,可以获得一个读取类内容的输入流，例：cl.getResourceAsStream(classname.replace(’.’,’/’)+".class")。

ClassWriter用于生成类：  
https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassWriterTest.java

在已有类中增加和删除一个字段(只有在常量的情况下可以设定初始值)：  
https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassVisitorAddTest.java

操作过程中，因为都是字节码，并不容易直观看到结果，可以借助TraceClassVisitor解决这个问题：  
https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/TraceVisitor.java


-----

Java 栈帧包含局部变量和操作数栈两部分。

操作码 0 用助记符号 NOP 表示,对应于不做任何操作的指令

字节代码指令可以分为两类:一小组指令,设计用来在局部变量和操作数栈之间传送值;其他一些指令仅用于操作数栈:它们从栈中弹出一些值,根据这些值计算一个结果,并将它压回栈中。

用来在局部变量和操作数栈之间传送值的指令：  
ILOAD, LLOAD, FLOAD, DLOAD 和 ALOAD 指令读取一个局部变量,并将它的值压到操作数栈中。它们的参数是必须读取的局部变量的索引 i。  
ILOAD 用于加载一个 boolean、byte、char、 short 或 int 局部变量。   
LLOAD、 FLOAD 和 DLOAD 分别用于加载 long、 float 或 double值。  
(LLOAD 和 DLOAD 实际加载两个槽 i 和 i+1，因为long和double一个slot放不下)  
ALOAD 用于加载任意非基元值,即对象和数组引用。  
与之对应,ISTORE、LSTORE、FSTORE、DSTORE 和 ASTORE 指令从操作数栈中弹出一个值,并将它存储在由其索引 i 指定的局部变量中。  
指令区分类型可确保不会执行非法转换

除上面指令外，其他字节代码指令都仅对操作数栈有效，以划分为以下类别：

用于处理栈上的值:  
POP 弹出栈顶部的值;DUP 压入顶部栈值的一个副本;SWAP 弹出两个值,并按逆序压栈;等

在操作数栈压入一个常量值:  
ACONST_NULL 压入 null,ICONST_0 压入int 值 0, FCONST_0 压入 0f, DCONST_0 压入 0d, BIPUSH b 压入字节值 b, SIPUSH s 压入 short 值 s, 
LDC cst 压入任意 int、 float、long、 double、 String 或 class常量 cst

从操作数栈弹出数值,计算并将结果压入栈中，（指令本身不带参数）：
xADD、xSUB、xMUL、xDIV 和 xREM 对应于+、-、* 、/ 和 % 运算,其中 x 为 I、L、F 或 D 之一。类似地,还有其他对应于<<、>>、>>>、|、&和^运算的指令,用于
处理 int 和 long 值
取负：ineg,lneg,fneg,dneg 
移位：ishl,lshr,iushr,lshl,lshr,lushr 
按位或：ior,lor 
按位与：iand,land 
按位异或：ixor,lxor 

类型转换,从栈中弹出一个值,将其转换为另一类型,并将结果压栈。对应 Java 中的类型转换表达式。I2F, F2D, L2D 等将数值由一种数值类型转换为另一种类型。i2l,i2f,i2d,l2f,l2d,f2d(放宽数值转换) i2b,i2c,i2s,l2i,f2i,f2l,d2i,d2l,d2f(缩窄数值转换)  
CHECKCAST t 将一个引用值转换为类型 t。

用于创建对象、锁定它们、检测它们的类型,等等。  
例如,NEW type 指令将一个 type 类型的新对象压入栈中(其中 type 是一个内部名)。
创建新数组：newarray,anewarray,multianwarray 

读或写一个字段的值:  
GETFIELD owner name desc 弹出一个对象引用,并将其 name 字段中的值压栈。PUTFIELD owner name desc 弹出一个值和一个对象引用,并将这个值存储在它的 name 字段中。在这两种情况下,该对象都必须是 owner 类型,它的字段必须为 desc 类型。GETSTATIC 和 PUTSTATIC 类似,用于静态字段。

-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Java_Type_Descriptor.md)
