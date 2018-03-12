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

类、字段、方法和方法参数注解,比如@Deprecated 或@Override,只要它们的保留策略不是 RetentionPolicy.SOURCE,它们就会被存储在编译后的类中。这一信息不是在运行时供字节代码指令使用,但是,如果保留策略是 RetentionPolicy.RUNTIME ,则可以通过反射 API 访问它。
例，对于加了@Deprecated的方法，使用javap -verbose aaa.class，在该方法中：

-----

      LineNumberTable:
        line 17: 0
        line 18: 8
        line 19: 16
        line 20: 22
        line 21: 27
        line 22: 32
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      34     0 globalReturnCode   Ljava/lang/String;
            0      34     1     d   Ljava/lang/Object;
            8      26     2  body   Lcom/aaaa/a/a/base/A;
      LocalVariableTypeTable:
        Start  Length  Slot  Name   Signature
            0      34     1     d   TT;
            8      26     2  body   Lcom/aaaa/a/a/base/A<TT;>;
    Deprecated: true
    Signature: #36                          // <T:Ljava/lang/Object;>(Ljava/lang/String;TT;)Lcom/aaaa/a/a/base/A;
    RuntimeVisibleAnnotations:
      0: #38()


-----

可以看到Deprecated：true，后面RuntimeVisibleAnnotations的0:指向的常量池位置:     
  #37 = Utf8               RuntimeVisibleAnnotations     
  #38 = Utf8               Ljava/lang/Deprecated;     

代码中的注解可以具有各种不同形式, 比如 @Deprecated 、@Retention(RetentionPolicy.CLASS)或@Task(desc="refactor", id=1)。但在内部,所有注解的形式都是相同的,由一种注解类型和一组名称/值对规定,其中的取值仅限于如下几种:1.基元,String 或 Class 值;2.枚举值;3.注解值(注意,一个注解中可以包含其他注解,甚至可以包含注解数组。因此,注解可能非常复杂);4.上述值的数组。

对注解访问的类都是AnnotationVisitor的子类。它的方法用于访问一个注解的名称/值对。AnnotationVisitor的visit用于基元、String 和 Class 值(后者用 Type 对象表示);visitEnum用于枚举;visitAnnotation用于注解;visitArray用于访问数组，和上一段的四种类型相对应，这四个方法间没有顺序要求。

javac -g 编译的类中包含了其源文件的名字、源代码行编号与字节代码指令之间的映射、源代码中局部变量名与字节代码中局部变量槽之间的映射。当这一可选信息可用时,会在调试器中和异常栈轨迹中使用它们。见上面的LineNumberTable部分。一个给定行号可以出现在几个对中。这是因为,对于出现在一个源代码行中的表达式,其
在字节代码中的相应指令可能不是连续的。

调试信息用 ClassVisitor 和 MethodVisitor 类的三个方法访问：
1.源文件名用 ClassVisitor 类的 visitSource 方法访问;
2.源代码行号与字节代码指令之间的映射用 MethodVisitor 类的 visitLineNumber方法访问,每次访问一对;
3.源代码中局部变量名与字节代码中局部变量槽之间的映射用 MethodVisitor 类的visitLocalVariable 方法访问,每次访问一个多元组。

-----

下面部分没有经过测试，因为暂时用不上，没经过测试的代码全在unchecked包下

下面是另外一种方法：ASM 树 API。树 API 通常用于那些不能由核心 API 一次实现的转换。     
用于生成和转换已编译 Java 类的 ASM 树 API 是基于 ClassNode 类的，其中包含一些信息和各种Node，如FieldNode和MethodNode。用树 API 生成类的过程就是:创建一个 ClassNode 对象,并初始化它的字段。使用树 API 生成类时,需要多花费大约 30%的时间,占用的内存也多于使用核心 API。但可以按任意顺序生成类元素,这在一些情况下可能非常方便。添加和删除类就是在 ClassNode 对象的 fields 或 methods 列表中添加或删除元素。

方法的 ASM 树 API 是基于 MethodNode的。instructions 字段是一个指令列表,InsnList类型。InsnList 是一个由指令组成的双向链表, AbstractInsnNode 中有两个属性AbstractInsnNode prev和AbstractInsnNode next保持指向的引用。书上重点强调了：一个 AbstractInsnNode 对象在一个指令列表中最多出现一次。
一个 AbstractInsnNode 对象不能同时属于多个指令列表。一个结果是:如果一个 AbstractInsnNode 属于某个列表,要将它添加到另一列表,必须先将其从原列表中删除。另一结果是:将一个列表中的所有元素都添加到另一个列表中,将会清空第一个列表。这些其实都是在强调一点，AbstractInsnNode对象唯一。AbstractInsnNode 类是表示字节代码指令的抽象类。已有子类的命名方式 Xxx InsnNode 类,对应于 MethodVisitor 接口的 visitXxx Insn 方法,而且其构造方式完全相同。例如,VarInsnNode 类对应于 visitVarInsn 方法。标记与帧,还有行号,尽管它们并不是指令,但也都用 AbstractInsnNode 类的子类表示,即 LabelNode 、 FrameNode 和 LineNumberNode 类。这样就允许将它们恰好插在列表中对应的真实指令之前,与核心 API 中一样(在核心 API 中,就是恰在相应的指令之前访问标
记和帧)。因此,很容易使用 AbstractInsnNode 类提供的 getNext 方法找到跳转指令的目标:这是目标标记之后第一个是真正指令的 AbstractInsnNode。另一个结果是:与核心 API一样,只要标记保持不变,删除指令并不会破坏跳转指令。
用树 API 转换方法只需要修改一个 MethodNode 对象的字段,特别是 instructions 列表。尽管这个列表可以采用任意方式修改,但常见做法是通过迭代修改。事实上,与通用ListIterator 约定不同,InsnList 返回的 ListIterator 支持许多并发列表修改,这些修改与对 Iterator.next 的调用交织在一起。多线程并发是不受支持的(没看懂)。如果需要在一个列表的指令 i 之后插入几条指令,那另一种修改指令列表的常见做法是将这些新指令插入一个临时指令列表中,再在一个步骤内将这个临时列表插到主列表中。逐条插入指令也是可行的,但却非常麻烦,因为必须在每次插之后更新插入点。

到目前为止,我们看到的所有方法转换都是局部的,甚至有状态的转换也是如此,所谓“局部”是指,一条指令 i 的转换仅取决于与 i 有固定距离的指令。但还存在一些全局转换,在这种转换中,指令 i 的转换可能取决于与 i 有任意距离的指令。对于这些转换,树 API 真的很有帮助,也就是说,使用核心 API 实现它们将会非常非常复杂。

MethodNode 类扩展了 MethodVisitor 类,还提供了两个accept 方法,它以一个 MethodVisitor 或一个 ClassVisitor 为参数。accept 方法基于MethodNode 字段值生成事件,而 MethodVisitor 方法执行逆操作,即根据接收到的事件设定 MethodNode 字段。

ASM代码分析的两个重要类型：
数据流分析包括:对于一个方法的每条指令,计算其执行帧的状态。这一状态可能采用一种多少有些抽象的方式来表示。例如,引用值可能用一个值来表示,可以每个类一个值,可以是{null, 非 null,可为 null}集合中的三个可能值表示,等等。
控制流分析包括计算一个方法的控制流图,并对这个图进行分析。控制流图中的节点为指令,如果指令 j 可以紧跟在 i 之后执行,则图的有向边将连接这两条指令 i→j。

有两种类型的数据流分析可以执行:
正向分析是指对于每条指令,根据执行帧在执行此指令之前的状态,计算执行帧在这一指令之后的状态。
反向分析是指对于每条指令,根据执行帧在执行此指令之后的状态,计算执行帧在这一指令之前的状态。

正向数据流分析的执行是对于一个方法的每个字节代码指令,模拟它在其执行帧上的执行,通常包括:从栈中弹出值,合并,将结果压入栈中。这看起来似乎就是解释器或 Java 虚拟机做的事情,但事实上,它是完全不同的,因为其目标是对于所有可能出现的参数值,模拟一个方法中的所有可能执行路径。
控制流分析的基础是方法的控制流图。

用于代码分析的 ASM API 在 org.objectweb.asm.tree.analysis 包中。由包的名字可以看出,它是基于树 API 的。事实上,这个包提供了一个进行正向数据流分析的框架。整体数据流分析算法、将适当数量的值从栈中弹出和压回栈中的任务仅实现一次,用于Analyzer 和 Frame 类中的所有内容。合并值的任何和计算值集并集的任务需要自己通过 Interpreter 和 Value 抽象类的子类实现。尽管框架的主要目的是执行数据流分析,但 Analyzer 类也可构造所分析方法的控制流图。可以重写这个类的 newControlFlowEdge 和 newControlFlowExceptionEdge 方法,它们默认情况下不做任何事情。其结果可用于进行控制流分析。

unchecked/AnalyzerTest.java

控制流分析可以有许多应用。一个简单的例子就是计算方法的“圈复杂度”。这一度量定义为控制流图的边数减去节点数,再加上 2。例如,checkAndSetT 方法的控制流图的圈复杂度为 11-12+2=1。
补充计算圈复杂度的两个公式：
计算公式1：V(G)=e-n+2p。其中，e表示控制流图中边的数量，n表示控制流图中节点的数量，p图的连接组件数目（图的组件数是相连节点的最大集合）。因为控制流图都是连通的，所以p为1.
计算公式2：V(G)=区域数=判定节点数+1。其实，圈复杂度的计算还有更直观的方法，因为圈复杂度所反映的是“判定条件”的数量，所以圈复杂度实际上就是等于判定节点的数量再加上1，也即控制流图的区域数。

树 API 没有提供对泛型的任何支持!事实上,它用签名表示泛型,但却没有提供与 SignatureVisitor 对应的 SignatureNode 类。

作为被编译类来源的源文件存储在 ClassNode 中的 sourceFile 字段中。关于源代码行号的信息存储在 LineNumberNode 对象中,它的类继承自 AbstractInsnNode。在核心 API中,关于行号的信息是与指令同时受访问的,与此类似,LineNumberNode 对象是指令列表的一部分。最后,源局部变量的名字和类型存储在 MethodNode 的 localVariables 字段中,它是 LocalVariableNode 对象的一个列表。

向后兼容部分以我目前的目的来说没有什么要求，先简单抄这了：
如果使用树 API 编写一个类生成器,那就不需要遵循什么规则(和核心 API 一样)。可以用任意构造器创建 ClassNode 和其他元素,可以使用这些类的任意方法。
另一方面,如果要用树 API 编写类分析器或类适配器,也就是说,如果使用 ClassNode或其他直接或间接地通过 ClassReader.accept()填充的类似类,或者如果重写这些类中的一个则必须注意。总之就是版本的问题的一些原则，核心API和树API都有。

规则 1:要为 ASM X 编写一个 ClassVisitor 子类,就以这个版本号为参数,调用ClassVisitor 构造器,在这个版本的 ClassVisitor 类中,绝对不要重写或调用弃用的方法(或者将在之后版本引入的方法)。

规则 2:不要使用访问器的继承,而要使用委托(即访问器链)。一种好的做法是让你的访问器类在默认情况为 final 的,以确保这一特性。(此规则有两个例外，总之自己确定就好)

规则 3:要用 ASM 版本 X 的树 API 编写类分析器或适配器,则使用以这一确切版本为参数的构造器创建 ClassNode(而不是使用没有参数的默认构造器)。

规则 4:要用 ASM 版本 X 的树 API 编写一个类分析器或适配器,使用别人创建的ClassNode,在以任何方式使用这个 ClassNode 之前,都要以这个确切版本号为参数,调用它的 check()方法。

还有一些特定情况的规则比如继承规则就是对上面规则的应用，就不抄了。

asm.util 和 asm.commons 中的类都有两个构造函数变体:一个有 ASM 版本参数,一个没有。
如果只是希望像 asm.util 中的 ASMifier、Textifier 或 CheckXxx Adapter 类或者asm.commons 包中的任意类一样,加以实例化和应用,那可以用没有 ASM 版本参数的构造器
来实例化它们。也可以使用带有 ASM 版本参数的构造器,那就会不必要地将这些组件限制于特定的 ASM 版本(而使用无参数构造器相当于在说“使用最新的 ASM 版本”)。这就是为什么使用 ASM 版本参数的构造器被声明为 protected。
另一方面,如果希望重写 asm.util 中的 ASMifier、 Textifier 或 CheckXxx Adapter类或者 asm.commons 包中的任意类,那适用规则 1 和 2。具体来说,你的构造器必须以你希望用作参数的 ASM 版本来调用 super(...)。
最后,如果希望使用或重写 asm.tree.analysis 中的 Interpreter 类或其子类,必须做出同样的区分。还要注意,在使用这个分析包之前,创建一个 MethodNode 或者从别人那里获取一个,那在将这一代码传送给 Analyzer 之前必须使用规则 3 和 4。

-----


[edit](https://github.com/saaavsaaa/saaavsaaa.github.io/edit/master/aaa/Java_Type_Descriptor.md)
