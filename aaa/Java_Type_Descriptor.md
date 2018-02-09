ASM书上讲的并不是很仔细，语句经常不通顺，例子经常缺点东西，我整理了一下，例子一部分自己做的，一部分在原例子上改的

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

[ClassReader用于分析已存在的类](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassReaderTest.java) 

注意,构建ClassReader实例的方式有多种。必须读取的类可以用名字指定,也可以像字母数组或InputStream一样用值来指定。利用ClassLoader的getResourceAsStream方法,可以获得一个读取类内容的输入流，例：cl.getResourceAsStream(classname.replace(’.’,’/’)+".class")。

[ClassWriter用于生成类](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassWriterTest.java) 

[在已有类中增加和删除一个字段(只有在常量的情况下可以设定初始值)](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassVisitorAddTest.java) 

[操作过程中，因为都是字节码，并不容易直观看到结果，可以借助TraceClassVisitor解决这个问题](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/TraceVisitor.java) 

[部分字节码指令](https://saaavsaaa.github.io/aaa/Java_Byte_Code.html) 

用于生成和转换已编译方法的 ASM API 是基于 MethodVisitor 抽象类的,它由 ClassVisitor 的 visitMethod 方法返回。除了一些与注释和调试信息有关的方法之外,这个类为每个字节代码指令类别定义了一个方法,其依据就是这些指令的参数个数和类型。对于非抽象方法,如果存在注释和属性的话,必须首先访问它们,然后是该方
法的字节代码。对于这些方法,其代码必须按顺序访问,位于对 visitCode 的调用(有且仅有一个调用)与对 visitMaxs 的调用(有且仅有一个调用)之间。

visitAnnotationDefault  
( visitAnnotation | visitParameterAnnotation | visitAttribute )*  
[ visitCode  
( visitTryCatchBlock | visitLabel | visitFrame | visitXxxInsn |visitLocalVariable | visitLineNumber )*  
visitMaxs ]  
visitEnd

visitCode 和 visitMaxs 方法可用于检测该方法的字节代码在一个事件序列中的开始与结束。和类的情况一样,visitEnd 方法也必须在最后调用,用于检测一个方法在一个事件序列中的结束 。多个MethodVisitor实例是完全独立的，没有顺序关系。

方法与类相似，同样是借助ClassReader、ClassWriter，不同的是返回MethodVisitor 

用ClassWriter创建一个方法时需要计算方法的局部变量与操作数栈部分的大小，ClassWriter提供了辅助方法   

    在使用 new ClassWriter(0)时,不会自动计算任何东西。必须自行计算帧(StackMapFrame)、局部变量与操作数栈的大小。使用visitMaxs手动传入局部标量和操作数栈的大小   
    在使用 new ClassWriter(ClassWriter.COMPUTE_MAXS)时,将为你计算局部变量与操作数栈部分的大小，但不会计算StackMapFrame。visitMaxs仍然需要调用,但可以使用任何参数:它们将被忽略并重新计算   
    在 new ClassWriter(ClassWriter.COMPUTE_FRAMES)时,一切都是自动计算。不再需要调用 visitFrame,但仍然必须调用 visitMaxs(参数将被忽略并重新计算)

COMPUTE_MAXS 选项使 ClassWriter 的速度降低 10%,而使用 COMPUTE_FRAMES 选项则使其降低一半。用 visitFrame(F_NEW, nLocals, locals, nStack, stack)访问未压缩帧可以执行压缩(删掉某些栈映射帧来压缩栈帧长度),其中的nLocals 和 nStack 是局部变量的个数和操作数栈的大小,locals和stack 是包含相应类型的数组

当计算帧需要两个指定类的公共超类时，默认ClassWriter会用getCommonSuperClass方法将这两个类加载并在存在security manager的情况下使用反射（Class.forName）。这时如果正在生成几个相互引用的类,可能会出现问题,被引用的类可能不存在。在这种情况下,可以通过重写 getCommonSuperClass 方法来解决问题。

[ClassWriter例子(增加方法及指令说明，修改方法，包括方法的跟踪与验证)](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassMethodVisitor.java)      

[补充：修改方法中指令的两个例子](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/PatternMethodAdapter.java)  

[根据 visitFrame 中访问的帧,计算每条指令之前的栈映射帧,在实践中并不建议使用这一方法,因为它的效率要远低于使用 COMPUTE_MAXS，继承自AnalyzerAdapter的两个例子AddTimerMethodAdapter2、AddTimerMethodAdapter3，不过要注意，书上写，要分析这个字节码的时候ClassReader的accept方法的第二个参数要传EXPAND_FRAMES，而原本的AddTimerMethodAdapter是无所谓的。](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassMethodVisitor.java) 

LocalVariablesSorter 方法适配器将一个方法中使用的局部变量按照它们在这个方法中的出现顺序重新进行编号。例如,在一个有两个参数的方法中,第一个被读取或写入且索引大于或等于 3 的局部变量(前三个局部变量对应于 this 及两个方法参数,因此序号不会变化)被赋予索引 3,第二个被赋予索引 4,以此类推。在向一个方法中插入新的局部变量时,这个适配器很有用。没有这个适配器,就需要在所有已有局部变量之后添加新的局部变量,但遗憾的是,在 visitMaxs 中,要直到方法的末尾处才能知道这些局部变量的编号。

[LocalVariablesSorter.newLocal生成局部变量例子(AddTimerLocalAdapter)](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/ClassMethodVisitor.java)    

-----



[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Java_Type_Descriptor.md)
