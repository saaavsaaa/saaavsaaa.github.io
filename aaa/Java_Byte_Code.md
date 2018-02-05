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

调用一个方法或一个构造器。指令弹出栈的值的个数等于其方法参数个数加 1(用于目标对象),并压回方法调用的结果。
INVOKEVIRTUAL owner name desc 调用在类 owner 中定义的 name 方法,其方法描述符为 desc。
INVOKESTATIC 用于静态方法,INVOKESPECIAL 用于私有方法和构造器,INVOKEINTERFACE 用于接口中定义的方法。
最后,对于 Java 7 中的类,INVOKEDYNAMIC 用于新动态方法调用机制。

用于读写数组中的值:  
xALOAD 指令弹出一个索引和一个数组,并压入此索引处数组元素的值。xASTORE 指令弹出一个值、一个索引和一个数组,并将这个值存储在该数组的这一索引处。这里的 x 可以是 I、L、F、D 或 A,还可以是 B、C 或 S。

无条件地或者在某一条件为真时跳转到一条任意指令。用于编译 if、for、do、while、break 和 continue 指令。
例如,IFEQ label 从栈中弹出一个int 值,如果这个值为 0,则跳转到由这个 label 指定的指令处(否则,正常执行下一
条指令)。还有许多其他跳转指令,比如 IFNE 或 IFGE。
TABLESWITCH 和LOOKUPSWITCH 对应于 switch Java 指令;TABLESWITCH会列出最大值和最小值中的所有值，使用索引访问，效率相对比较高，但相对会浪费空间，代码中没有的分支值会对应到default；当分支特别多时，编译器会衡量空间和时间的消耗，选择LOOKUPSWITCH时，栈顶的值会和lookupswitch里的每个值进行比较，JVM会在列表中查找匹配的元素，确定要跳转的分支。

返回 :
xRETURN 和 RETURN 指令用于终止一个方法的执行,并将其结果返回给调用者。RETURN 用于返回 void 的方法,xRETURN 用于其他方法。

例：

    IFLT label  
    ALOAD 0  
    ILOAD 1  
    PUTFIELD pkg/Bean f I  
    GOTO end  
    label:  
    NEW java/lang/IllegalArgumentException  
    DUP  
    INVOKESPECIAL java/lang/IllegalArgumentException <init> ()V  
    ATHROW  
    end:  
    RETURN  

IFLT 指令从栈中弹出值,并将它与 0 进行比较。如果它小于(LT)0,则跳转到由 label 标记指定的指令,否则不做任何事情

NEW 指令创建一个异常对象,对其进行默认初始化,并将它压入操作数栈中。DUP 指令在栈中重复这个对象的引用的副本入栈，为了在INVOKESPECIAL指令将对象引用出栈后还可以使用这个对象的引用。INVOKESPECIAL 指令弹出这两个副本之一,并对其调用异常构造器。最后,ATHROW 指令弹出剩下的副本,并将它作为异常抛出

    public static void sleep(long d) {
        try {
        Thread.sleep(d);
        } catch (InterruptedException e) {
        e.printStackTrace();
        }
    }

可被编译为  

    TRYCATCHBLOCK try catch catch java/lang/InterruptedException  
    try:  
    LLOAD 0  
    INVOKESTATIC java/lang/Thread sleep (J)V  
    RETURN  
    catch:  
    INVOKEVIRTUAL java/lang/InterruptedException printStackTrace ()V  
    RETURN  


TRYCATCHBLOCK 行指定了一个异常处理器,处理器开始于 catch 标记,用于处理一些异常,这些异常的类是 InterruptedException的子类。这意味着,如果在 try 和 catch 之间抛出了这样一个异常,栈将被清空,异常被压入这个空栈中,执行过程在 catch 处继续。

-----

[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Java_Byte_Code.md)
