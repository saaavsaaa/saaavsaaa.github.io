ASM书上讲的并不是很仔细，或者是翻译的问题，语句经常不通顺，例子经常缺点东西或者是错的，我整理了一下，例子一部分自己做的，一部分在原例子上改的。

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

AdviceAdapter 方法适配器是一个抽象类,可用于在一个方法的开头以及恰在任意 RETURN 或 ATHROW指令之前插入代码。它的主要好处就是对于构造器也是有效的,在构造器中,不能将代码恰好插入到构造器的开头,而是插在对超构造器的调用之后。事实上,这个适配器的大多数代码都专门用于检测对这个超构造器的调用。

-----

类型变量是在类、接口、方法和构造器中用作类型的非限定标识符   
类型变量可以声明为泛化类声明、泛化接口声明、泛化方法声明和泛化构造器声明中的类型参数

诸如 List<E>之类的泛型类,以及使用它们的类,包含了有关它们所声明或使用的泛型的信息。这一信息不是由字节代码指令在运行时使用,但可通过反射 API 访问。它还可以供编译器使用,以进行分离编译。
 
泛型的信息没有存储在类型或方法描述符中,而是保存在称为类型、方法和类签名的类似构造中。在涉及泛型时,除了描述符之外,这些签名也会存储在类、字段和方法声明中(泛型不会影响方法的字节代码:编译器用它们执行静态类型检查,但会在必要时重新引入类型转换,就像这些方法未被使用一样进行编译)。

与类型和方法描述符不同,类型签名的语法非常复杂,因为泛型本身可递归(例如：List<List<E>>)。书上说可参阅《Java 虚拟机规范》，然而无论是虚拟机规范里并没找到多少相关内容，语言规范里倒是不少。

类型签名的一些规则：
TypeSignature: Z | C | B | S | I | F | J | D | FieldTypeSignature  ： 类型签名是一个基元类型描述符或者字段类型签名   

FieldTypeSignature: ClassTypeSignature | \[ TypeSignature | TypeVar  ： 一个字段类型签名定义为一个类类型签名、数组类型签名或类型变量    

ClassTypeSignature: L Id ( / Id )* TypeArgs? ( . Id TypeArgs? )* ;  ：类类型签名:类类型描述符,在主类名之后或者内部类名之后的尖括号中可能带有类型参数(以点为前缀)     

其他定义了类型参数和类型变量:
TypeArgs: < TypeArg+ >   
TypeArg: * | ( + | - )? FieldTypeSignature   
TypeVar: T Id ;   

注意,一个类型参数可能是一个完整的字段类型签名,带有它自己的类型参数:因此,类型签名可能非常复杂

|Java 类型 | 对应的类型签名|
|:-:|:-:|
|List<E>  | Ljava/util/List<TE;>;|
|List<?> | Ljava/util/List<*>;|
|List<? extends Number> | Ljava/util/List<+Ljava/lang/Number;>;|
|List<? super Integer> | Ljava/util/List<-Ljava/lang/Integer;>;|
|List<List<String>[]> | Ljava/util/List<[Ljava/util/List<Ljava/lang/String;>;>;|
|HashMap<K, V>.HashIterator<K> | Ljava/util/HashMap<TK;TV;>.HashIterator<TK;>;|

类型签名扩展了类型描述符，还包含了该方法所抛出异常的签名,前面带有^前缀,还可以在尖括号之间包含可选的形式类型参数:
TypeParams? ( TypeSignature* ) ( TypeSignature | V ) Exception*   
Exception: ^ClassTypeSignature | ^TypeVar   
TypeParams: < TypeParam+ >    
TypeParam: Id : FieldTypeSignature? ( : FieldTypeSignature )*     
例如以以类型变量 T 为参数的泛型静态方法:static <T> Class<? extends T> m (int n)，它的方法签名:<T:Ljava/lang/Object;>(I)Ljava/lang/Class<+TT;>;     
不要将类类型签名与类签名相混淆,类签名被定义为其超类的类型签名。后面跟所有接口的类型签名,以及可选的形式类型参数:ClassSignature: TypeParams? ClassTypeSignature ClassTypeSignature*。例如 , 个被声明为 C<E> extends List<E> 的类的类签名就是<E:Ljava/lang/Object;>Ljava/util/List<TE;>;。


SignatureVisitor可用于生成和转换签名，可以访问类型签名、方法签名和类签名。
其中可用于访问类签名的方法有：visitBaseType、visitTypeVariable、visitArrayType、visitClassType、visitInnerClassType、visitTypeArgument、visitTypeArgument和visitEnd。其中visitArrayType和visitTypeArgument因为类型签名是可递归的，所以返回SignatureVisitor。这几个方法的调用顺序：
visitBaseType | visitArrayType | visitTypeVariable |( visitClassType visitTypeArgument*( visitInnerClassType visitTypeArgument* )* visitEnd ) )

用于访问方法签名的方法：( visitFormalTypeParameter visitClassBound visitInterfaceBound* )* visitParameterType* visitReturnType visitExceptionType*

用于访问类签名的方法：( visitFormalTypeParameter visitClassBound? visitInterfaceBound* )* visitSuperClass visitInterface*

这些方法大多返回一个 SignatureVisitor:它是准备用来访问类型签名的。注意,不同于 ClassVisitor 返 回 的 MethodVisitors ,SignatureVisitor返回的
SignatureVisitors 不得为 null,而且必须顺序使用:在完全访问一个嵌套签名之前,不得访问基类访问器的任何方法。

与类和方法的Reader、Writer一样，signatureReader 组件分析一个签名,并针对一个给定的签名访问器调用适当的访问方法;SignatureWriter 组件基于它接收到的方法调用生成一个签名。[改签名的例子](https://github.com/saaavsaaa/warn-report/blob/master/src/main/java/code/visit/SignatureAdapter.java)  

-----

Annotation部分：


-----


[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Java_Type_Descriptor.md)
