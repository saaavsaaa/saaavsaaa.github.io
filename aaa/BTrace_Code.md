xxd一个编译后的用于单元测试的class脚本就可以发现，这些脚本都是Btrace自己编译的，魔数位是0xbacecaca。因为在开发环境调试方便，我就直接用的com/sun/btrace/compiler/Compiler.java的main方法编译脚本了。

/home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/BTraceClassWriter.instrument
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/InstrumentUtils.accept
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/BTraceClassReader.accept     
--> btrace-asm-5.2.jar!/com/sun/btrace/org/objectweb/asm/ClassReader.accept     
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/BTraceClassReader.accept
--> btrace-asm-5.2.jar!/com/sun/btrace/org/objectweb/asm/ClassReader.accept -> b     
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/Instrumentor.visitMethod
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/InstrumentingMethodVisitor.ctor -> initLocals [load被拦截方法参数类型，这里isInstance是true，加载的本地变量算上变量所有者owner（自身实例）=参数个数+1，但是argSize由于循环增加时对owner加了两次于是=参数个数+2]

<<< Instrumentor.visitMethod : methodVisitor = new TemplateExpanderVisitor(methodVisitor, mHelper, className, name, desc); [ctor : this.expanders.add(new MethodTrackingExpander(MethodID.getMethodId(className, methodName, desc), mHelper)); this.asm = new Assembler(mv, mHelper);]     
循环脚本（脚本参数中本来Throw我是放在被拦截参数前面的，编译后读出来就在后面了，这一点要确认下）的拦截方法
each -> instrumentorFor[**这个方法比较重要，它根据location = @Location(Kind.?)创建处理对应情况的MethodVisitor**(既然是循环，多个返回是如何处理的？)]
<<< Instrumentor.visitMethod [new一个MethodVisitor返回]:

-----

        return new MethodVisitor(Opcodes.ASM5, methodVisitor) {
            @Override
            public AnnotationVisitor visitAnnotation(String annoDesc, boolean visible) {
                for (OnMethod om : annotationMatchers) {
                    String extAnnoName = Type.getType(annoDesc).getClassName();
                    String annoName = om.getMethod();
                    if (om.isMethodRegexMatcher()) {
                        try {
                            if (extAnnoName.matches(annoName)) {
                                mv = instrumentorFor(om, mv, mHelper, mAccess, name, desc);
                            }
                        } catch (PatternSyntaxException pse) {
                            reportPatternSyntaxException(extAnnoName);
                        }
                    } else if (annoName.equals(extAnnoName)) {
                        mv = instrumentorFor(om, mv, mHelper, mAccess, name, desc);
                    }
                }
                return mv.visitAnnotation(annoDesc, visible);
            }
        };
    }

-----

--> btrace-asm-5.2.jar!/com/sun/btrace/org/objectweb/asm/ClassReader.accept -> b  
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/instr/MethodInstrumentor.visitCode
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/Instrumentor.visitMethodPrologue
--> /home/aaa/Github/btrace/src/share/classes/com/sun/btrace/runtime/instr/ErrorReturnInstrumentor.visitMethodPrologue [因为我跑的这个是Error类型的] -> visitTryCatchBlock     
---> 
















脚本被Btrace编译后读入：
        File f = new File(path);
        InputStream is = new FileInputStream(f)

        while ((read = is.read(buffer)) > 0) {
            byte[] newresult = new byte[result.length + read];
            System.arraycopy(result, 0, newresult, 0, result.length);
            System.arraycopy(buffer, 0, newresult, result.length, read);
            result = newresult;
        }
 
这里的result做参数(traceData)来创建BTraceProbe :

        BTraceProbe bcn = factory.createProbe(traceData);
        
createProbe:

        int mgc = ((code[0] & 0xff) << 24) | ((code[1] & 0xff) << 16) | ((code[2] & 0xff) << 8) | (( & 0xff));

magic:class是一组以8位字节(byte)为基础单位的二进制流。在class文件开头的四个字节， 存放着class文件的魔数， 这个魔数是class文件的标志，他是一个固定的值： 0XCAFEBABE 。 也就是说他是判断一个文件是不是class格式的文件的标准， 如果开头四个字节不是0XCAFEBABE， 那么就说明它不是class文件， 不能被JVM识别。（可参考《深入理解Java虚拟机JVM高级特性与最佳实践》第六章）

这句代码是要将4个字节合成一个值与BTraceProbePersisted.MAGIC值进行比较。因为byte转为int，int会用32位中的低8位保存这个数，所以前3个字节分别进行了左位移。其中对每个字节和0xff的位与运算是因为：计算机中都是用的补码，因为计算机只能做加减运算，为了在运算中保持符号位的正确性所以使用的是补码。而这条语句一旦开始执行code[0]就会被表示为int类型，变为32位的，如果不做处理那么高的24位会补1导致补码值不正确，而先做&0xff运算，原来是0的还是0，是1的还是1，就保证了数据的正确性。(每个字节表示数的取值范围：因为-0的符号没有意义,所以符号位是1，其余都是0的补码可以用来表示-127-1，因为负数的补码符号位为1，其余位为该数绝对值的原码按位取反，然后加1)。code如果是我们自己编译的class文件读出来的byte数组，code[0] code[1] code[2] code[3]指的就是magic这4个字节。为什么要这么说呢，因为如果我们传给btrace的是个.java文件，它自己会进行编译，编译的class格式也是参照了虚拟机的格式，但是其中魔数用的是0xbacecaca，版本号用的是VERSION = 1，见BTraceProbePersisted.write(DataOutputStream dos)方法。

        $ xxd ArgsDurationErrAAA.class 
        00000000: bace caca 0000 0001 0022 7472 6163 6573 
        ...

利用BTraceProbePersisted.MAGIC验证了加载的文件是自己的脚本class文件后，用去掉了魔数后的byte数组创建BTraceProbePersisted对象。

        BTraceProbePersisted bpp = new BTraceProbePersisted(this);
                try (DataInputStream dis = new DataInputStream(new ByteArrayInputStream(Arrays.copyOfRange(code, 4, code.length)))) {
                    bpp.read(dis);
                    return bpp;

read验证了一下版本，如果通过，调用了read_1：

        private void read_1(DataInputStream dis) throws IOException {
                delegate.setClassName(dis.readUTF());
                readServices(dis);
                readOnMethods(dis);
                readOnProbes(dis);
                readCallees(dis);
                readDataHolderClass(dis);
                readFullData(dis);
        }

虽然这里不是遵守的java虚拟机的规范，但是方式是类似的，按着严格的顺序和紧凑的排列读取编译结果。

readUTF方法先读取流的下两个字节，转为一个无符号的16位整数，对应BTraceProbePersisted.write中写版本后的dos.writeUTF(getClassName(true))，上面xxd命令已经显示了这两个字节是0022, 2 * 16 + 2 = 34。读出来的值utflen为34，也就是说后面34个字节是BTraceProbeSupport.origName，如果类没有重命名className也是它。

        while (count < utflen) {
            c = (int) bytearr[count] & 0xff;
            if (c > 127) break;
            count++;
            chararr[chararr_count++]=(char)c;
        }

如果有出现c > 127的情况，下面还有一个循环处理。最后转成字符串返回，这个字符串就是类全路径名。

readInt读入下面4个字节，读取逻辑和刚才是一样的，我这里没有Field，所以是0000 0000：

        private void readServices(DataInputStream dis) throws IOException {
                int num = dis.readInt();
                for (int i = 0; i < num; i++) {
                    delegate.addServiceField(dis.readUTF(), dis.readUTF());
                }
        }

readOnMethods:

            private void readOnMethods(DataInputStream dis) throws IOException {
                int num = dis.readInt();
                for (int i = 0; i < num; i++) {
                    OnMethod om = new OnMethod();

接着的4个字节标识有多少个方法，依次处理每个方法，我这里是一个0000 0001，完整的xxd结果，贴在最后了：

                om.setClazz(dis.readUTF());                   设置被拦截的类，这里是正则形式，例如：/.*\.OnMethodTest/，0012表示长度
                om.setMethod(dis.readUTF());                  被拦截的方法，@OnMethod注解中method的值，例如：method=args的args；会判断是否正则，是否是注解; 0004 6172 6773;
                om.setExactTypeMatch(dis.readBoolean());      下一个byte转为boolean，我这里是00，所以是false
                bom.setTargetDescriptor(dis.readUTF());       参考xxd结果时可能要注意下，00和40分开了，长度64，内容是脚本方法的参数和返回值：(Ljava/lang/Throwable;Ljava/lang/String;J[Ljava/lang/String;[I)V  --> Throwable, String, long, String[], int[]
                om.setTargetName(dis.readUTF());              被标记了@OnMethod的脚本方法本身的方法名
                om.setType(dis.readUTF());                    0000
                om.setClassNameParameter(dis.readInt());      readInt4个255拼成-1
                om.setDurationParameter(dis.readInt());       
                om.setMethodParameter(dis.readInt());         
                om.setReturnParameter(dis.readInt());
                om.setSelfParameter(dis.readInt());
                om.setTargetInstanceParameter(dis.readInt());
                om.setTargetMethodOrFieldParameter(dis.readInt());
                om.setMethodFqn(dis.readBoolean());            我猜应该是方法完全限定名
                om.setTargetMethodOrFieldFqn(dis.readBoolean());
                om.setSamplerKind(Sampled.Sampler.valueOf(dis.readUTF()));      
                om.setSamplerMean(dis.readInt());
                om.setLevel(dis.readBoolean() ? Level.fromString(dis.readUTF()) : null);

                Location loc = new Location();
                loc.setValue(Kind.valueOf(dis.readUTF()));      默认值Kind.ENTRY，这里是ERROR
                loc.setWhere(Where.valueOf(dis.readUTF()));     BEFORE
                loc.setClazz(dis.readBoolean() ? dis.readUTF() : null);
                loc.setField(dis.readBoolean() ? dis.readUTF() : null);
                loc.setMethod(dis.readBoolean() ? dis.readUTF() : null);
                loc.setType(dis.readBoolean() ? dis.readUTF() : null);
                loc.setLine(dis.readInt());

之后还有readDataHolderClass和readFullData等几个方法，逻辑类似，总之就是把用BTrace自己规则编译的脚本文件读进内存，保存在BTraceProbePersisted对象的实例中。


data的大概内容：

                public class traces/onmethod/ArgsDurationErrAAA {


                  @Lcom/sun/btrace/annotations/BTrace;()

                  // access flags 0x9
                  public static Lcom/sun/btrace/BTraceRuntime; runtime

                  // access flags 0x49
                  public static volatile I $btrace$$level = 0

                  // access flags 0x9
                  public static <clinit>()V
                    TRYCATCHBLOCK L0 L1 L1 java/lang/Throwable
                   L0
                    LDC Ltraces/onmethod/ArgsDurationErrAAA;.class
                    ACONST_NULL
                    ACONST_NULL
                    ACONST_NULL
                    ACONST_NULL
                    ACONST_NULL
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.forClass (Ljava/lang/Class;[Lcom/sun/btrace/shared/TimerHandler;[Lcom/sun/btrace/shared/EventHandler;[Lcom/sun/btrace/shared/ErrorHandler;[Lcom/sun/btrace/shared/ExitHandler;[Lcom/sun/btrace/shared/LowMemoryHandler;)Lcom/sun/btrace/BTraceRuntime;
                    PUTSTATIC traces/onmethod/ArgsDurationErrAAA.runtime : Lcom/sun/btrace/BTraceRuntime;
                    GETSTATIC traces/onmethod/ArgsDurationErrAAA.runtime : Lcom/sun/btrace/BTraceRuntime;
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.enter (Lcom/sun/btrace/BTraceRuntime;)Z
                    IFNE L2
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.leave ()V
                    RETURN
                   L2
                   FRAME SAME
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.start ()V
                    RETURN
                   L1
                   FRAME SAME1 java/lang/Throwable
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.handleException (Ljava/lang/Throwable;)V
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.leave ()V
                    RETURN
                    MAXSTACK = 6
                    MAXLOCALS = 0

                  // access flags 0x1
                  public <init>()V
                    ALOAD 0
                    INVOKESPECIAL java/lang/Object.<init> ()V
                    RETURN
                    MAXSTACK = 1
                    MAXLOCALS = 1

                  // access flags 0x9
                  public static args(Ljava/lang/Throwable;Ljava/lang/String;J[Ljava/lang/String;[I)V
                  @Lcom/sun/btrace/annotations/OnMethod;(clazz="/.*\\.OnMethodTest/", method="args", location=@Lcom/sun/btrace/annotations/Location;(value=Lcom/sun/btrace/annotations/Kind;.ERROR))
                    TRYCATCHBLOCK L0 L1 L1 java/lang/Throwable
                    GETSTATIC traces/onmethod/ArgsDurationErrAAA.runtime : Lcom/sun/btrace/BTraceRuntime;
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.enter (Lcom/sun/btrace/BTraceRuntime;)Z
                    IFNE L0
                    RETURN
                   L0
                   FRAME SAME
                    LDC "args"
                    INVOKESTATIC com/sun/btrace/BTraceUtils.println (Ljava/lang/Object;)V
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.leave ()V
                    RETURN
                   L1
                   FRAME SAME1 java/lang/Throwable
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.handleException (Ljava/lang/Throwable;)V
                    INVOKESTATIC com/sun/btrace/BTraceRuntime.leave ()V
                    RETURN
                    MAXSTACK = 1
                    MAXLOCALS = 6
                }

-----

        验证的话就是判断是否信任，是否需要验证，验证的内容包括基类中是不是包含Object之类，还有各种类型的方法的验证，比如构造函数<init>比如INVOKESPECIAL关键字等等，可参考BTraceProbePersisted.verifyBytecode。
























































-----

                $ xxd ArgsDurationErrAAA.class
                00000000: bace caca 0000 0001 0022 7472 6163 6573  ........."traces
                00000010: 2f6f 6e6d 6574 686f 642f 4172 6773 4475  /onmethod/ArgsDu
                00000020: 7261 7469 6f6e 4572 7241 4141 0000 0000  rationErrAAA....
                00000030: 0000 0001 0012 2f2e 2a5c 2e4f 6e4d 6574  ....../.*\.OnMet
                00000040: 686f 6454 6573 742f 0004 6172 6773 0000  hodTest/..args..
                00000050: 4028 4c6a 6176 612f 6c61 6e67 2f54 6872  @(Ljava/lang/Thr
                00000060: 6f77 6162 6c65 3b4c 6a61 7661 2f6c 616e  owable;Ljava/lan
                00000070: 672f 5374 7269 6e67 3b4a 5b4c 6a61 7661  g/String;J[Ljava
                00000080: 2f6c 616e 672f 5374 7269 6e67 3b5b 4929  /lang/String;[I)
                00000090: 5600 0461 7267 7300 00ff ffff ffff ffff  V..args.........
                000000a0: ffff ffff ffff ffff ffff ffff ffff ffff  ................
                000000b0: ffff ffff ff00 0000 044e 6f6e 6500 0000  .........None...
                000000c0: 0000 0005 4552 524f 5200 0642 4546 4f52  ....ERROR..BEFOR
                000000d0: 4501 0000 0100 0001 0000 0100 0000 0000  E...............
                000000e0: 0000 0000 0000 0000 0000 0003 9aca feba  ................
                000000f0: be00 0000 3300 2c01 0022 7472 6163 6573  ....3.,.."traces
                00000100: 2f6f 6e6d 6574 686f 642f 4172 6773 4475  /onmethod/ArgsDu
                00000110: 7261 7469 6f6e 4572 7241 4141 0700 0101  rationErrAAA....
                00000120: 0010 6a61 7661 2f6c 616e 672f 4f62 6a65  ..java/lang/Obje
                00000130: 6374 0700 0301 0023 4c63 6f6d 2f73 756e  ct.....#Lcom/sun
                00000140: 2f62 7472 6163 652f 616e 6e6f 7461 7469  /btrace/annotati
                00000150: 6f6e 732f 4254 7261 6365 3b01 0007 7275  ons/BTrace;...ru
                00000160: 6e74 696d 6501 001e 4c63 6f6d 2f73 756e  ntime...Lcom/sun
                00000170: 2f62 7472 6163 652f 4254 7261 6365 5275  /btrace/BTraceRu
                00000180: 6e74 696d 653b 0100 0e24 6274 7261 6365  ntime;...$btrace
                00000190: 2424 6c65 7665 6c01 0001 4903 0000 0000  $$level...I.....
                000001a0: 0100 083c 636c 696e 6974 3e01 0003 2829  ...<clinit>...()
                000001b0: 5601 0013 6a61 7661 2f6c 616e 672f 5468  V...java/lang/Th
                000001c0: 726f 7761 626c 6507 000d 0100 1c63 6f6d  rowable......com
                000001d0: 2f73 756e 2f62 7472 6163 652f 4254 7261  /sun/btrace/BTra
                000001e0: 6365 5275 6e74 696d 6507 000f 0100 0866  ceRuntime......f
                000001f0: 6f72 436c 6173 7301 00ed 284c 6a61 7661  orClass...(Ljava
                00000200: 2f6c 616e 672f 436c 6173 733b 5b4c 636f  /lang/Class;[Lco
                00000210: 6d2f 7375 6e2f 6274 7261 6365 2f73 6861  m/sun/btrace/sha
                00000220: 7265 642f 5469 6d65 7248 616e 646c 6572  red/TimerHandler
                00000230: 3b5b 4c63 6f6d 2f73 756e 2f62 7472 6163  ;[Lcom/sun/btrac
                00000240: 652f 7368 6172 6564 2f45 7665 6e74 4861  e/shared/EventHa
                00000250: 6e64 6c65 723b 5b4c 636f 6d2f 7375 6e2f  ndler;[Lcom/sun/
                00000260: 6274 7261 6365 2f73 6861 7265 642f 4572  btrace/shared/Er
                00000270: 726f 7248 616e 646c 6572 3b5b 4c63 6f6d  rorHandler;[Lcom
                00000280: 2f73 756e 2f62 7472 6163 652f 7368 6172  /sun/btrace/shar
                00000290: 6564 2f45 7869 7448 616e 646c 6572 3b5b  ed/ExitHandler;[
                000002a0: 4c63 6f6d 2f73 756e 2f62 7472 6163 652f  Lcom/sun/btrace/
                000002b0: 7368 6172 6564 2f4c 6f77 4d65 6d6f 7279  shared/LowMemory
                000002c0: 4861 6e64 6c65 723b 294c 636f 6d2f 7375  Handler;)Lcom/su
                000002d0: 6e2f 6274 7261 6365 2f42 5472 6163 6552  n/btrace/BTraceR
                000002e0: 756e 7469 6d65 3b0c 0011 0012 0a00 1000  untime;.........
                000002f0: 130c 0006 0007 0900 0200 1501 0005 656e  ..............en
                00000300: 7465 7201 0021 284c 636f 6d2f 7375 6e2f  ter..!(Lcom/sun/
                00000310: 6274 7261 6365 2f42 5472 6163 6552 756e  btrace/BTraceRun
                00000320: 7469 6d65 3b29 5a0c 0017 0018 0a00 1000  time;)Z.........
                00000330: 1901 0005 6c65 6176 650c 001b 000c 0a00  ....leave.......
                00000340: 1000 1c01 0005 7374 6172 740c 001e 000c  ......start.....
                00000350: 0a00 1000 1f01 000f 6861 6e64 6c65 4578  ........handleEx
                00000360: 6365 7074 696f 6e01 0018 284c 6a61 7661  ception...(Ljava
                00000370: 2f6c 616e 672f 5468 726f 7761 626c 653b  /lang/Throwable;
                00000380: 2956 0c00 2100 220a 0010 0023 0100 063c  )V..!."....#...<
                00000390: 696e 6974 3e0c 0025 000c 0a00 0400 2601  init>..%......&.
                000003a0: 000d 436f 6e73 7461 6e74 5661 6c75 6501  ..ConstantValue.
                000003b0: 0004 436f 6465 0100 0d53 7461 636b 4d61  ..Code...StackMa
                000003c0: 7054 6162 6c65 0100 1952 756e 7469 6d65  pTable...Runtime
                000003d0: 5669 7369 626c 6541 6e6e 6f74 6174 696f  VisibleAnnotatio
                000003e0: 6e73 0021 0002 0004 0000 0002 0009 0006  ns.!............
                000003f0: 0007 0000 0049 0008 0009 0001 0028 0000  .....I.......(..
                00000400: 0002 000a 0002 0009 000b 000c 0001 0029  ...............)
                00000410: 0000 0046 0006 0000 0000 0025 1202 0101  ...F.......%....
                00000420: 0101 01b8 0014 b300 16b2 0016 b800 1a9a  ................
                00000430: 0007 b800 1db1 b800 20b1 b800 24b8 001d  ........ ...$...
                00000440: b100 0100 0000 1e00 1e00 0e00 0100 2a00  ..............*.
                00000450: 0000 0700 021a 4307 000e 0001 0025 000c  ......C......%..
                00000460: 0001 0029 0000 0011 0001 0001 0000 0005  ...)............
                00000470: 2ab7 0027 b100 0000 0000 0100 2b00 0000  *..'........+...
                00000480: 0600 0100 0500 0000 0005 55ca feba be00  ..........U.....
                00000490: 0000 3300 3e01 0022 7472 6163 6573 2f6f  ..3.>.."traces/o
                000004a0: 6e6d 6574 686f 642f 4172 6773 4475 7261  nmethod/ArgsDura
                000004b0: 7469 6f6e 4572 7241 4141 0700 0101 0010  tionErrAAA......
                000004c0: 6a61 7661 2f6c 616e 672f 4f62 6a65 6374  java/lang/Object
                000004d0: 0700 0301 0023 4c63 6f6d 2f73 756e 2f62  .....#Lcom/sun/b
                000004e0: 7472 6163 652f 616e 6e6f 7461 7469 6f6e  trace/annotation
                000004f0: 732f 4254 7261 6365 3b01 0007 7275 6e74  s/BTrace;...runt
                00000500: 696d 6501 001e 4c63 6f6d 2f73 756e 2f62  ime...Lcom/sun/b
                00000510: 7472 6163 652f 4254 7261 6365 5275 6e74  trace/BTraceRunt
                00000520: 696d 653b 0100 0e24 6274 7261 6365 2424  ime;...$btrace$$
                00000530: 6c65 7665 6c01 0001 4903 0000 0000 0100  level...I.......
                00000540: 083c 636c 696e 6974 3e01 0003 2829 5601  .<clinit>...()V.
                00000550: 0013 6a61 7661 2f6c 616e 672f 5468 726f  ..java/lang/Thro
                00000560: 7761 626c 6507 000d 0100 1c63 6f6d 2f73  wable......com/s
                00000570: 756e 2f62 7472 6163 652f 4254 7261 6365  un/btrace/BTrace
                00000580: 5275 6e74 696d 6507 000f 0100 0866 6f72  Runtime......for
                00000590: 436c 6173 7301 00ed 284c 6a61 7661 2f6c  Class...(Ljava/l
                000005a0: 616e 672f 436c 6173 733b 5b4c 636f 6d2f  ang/Class;[Lcom/
                000005b0: 7375 6e2f 6274 7261 6365 2f73 6861 7265  sun/btrace/share
                000005c0: 642f 5469 6d65 7248 616e 646c 6572 3b5b  d/TimerHandler;[
                000005d0: 4c63 6f6d 2f73 756e 2f62 7472 6163 652f  Lcom/sun/btrace/
                000005e0: 7368 6172 6564 2f45 7665 6e74 4861 6e64  shared/EventHand
                000005f0: 6c65 723b 5b4c 636f 6d2f 7375 6e2f 6274  ler;[Lcom/sun/bt
                00000600: 7261 6365 2f73 6861 7265 642f 4572 726f  race/shared/Erro
                00000610: 7248 616e 646c 6572 3b5b 4c63 6f6d 2f73  rHandler;[Lcom/s
                00000620: 756e 2f62 7472 6163 652f 7368 6172 6564  un/btrace/shared
                00000630: 2f45 7869 7448 616e 646c 6572 3b5b 4c63  /ExitHandler;[Lc
                00000640: 6f6d 2f73 756e 2f62 7472 6163 652f 7368  om/sun/btrace/sh
                00000650: 6172 6564 2f4c 6f77 4d65 6d6f 7279 4861  ared/LowMemoryHa
                00000660: 6e64 6c65 723b 294c 636f 6d2f 7375 6e2f  ndler;)Lcom/sun/
                00000670: 6274 7261 6365 2f42 5472 6163 6552 756e  btrace/BTraceRun
                00000680: 7469 6d65 3b0c 0011 0012 0a00 1000 130c  time;...........
                00000690: 0006 0007 0900 0200 1501 0005 656e 7465  ............ente
                000006a0: 7201 0021 284c 636f 6d2f 7375 6e2f 6274  r..!(Lcom/sun/bt
                000006b0: 7261 6365 2f42 5472 6163 6552 756e 7469  race/BTraceRunti
                000006c0: 6d65 3b29 5a0c 0017 0018 0a00 1000 1901  me;)Z...........
                000006d0: 0005 6c65 6176 650c 001b 000c 0a00 1000  ..leave.........
                000006e0: 1c01 0005 7374 6172 740c 001e 000c 0a00  ....start.......
                000006f0: 1000 1f01 000f 6861 6e64 6c65 4578 6365  ......handleExce
                00000700: 7074 696f 6e01 0018 284c 6a61 7661 2f6c  ption...(Ljava/l
                00000710: 616e 672f 5468 726f 7761 626c 653b 2956  ang/Throwable;)V
                00000720: 0c00 2100 220a 0010 0023 0100 063c 696e  ..!."....#...<in
                00000730: 6974 3e0c 0025 000c 0a00 0400 2601 0004  it>..%......&...
                00000740: 6172 6773 0100 4028 4c6a 6176 612f 6c61  args..@(Ljava/la
                00000750: 6e67 2f54 6872 6f77 6162 6c65 3b4c 6a61  ng/Throwable;Lja
                00000760: 7661 2f6c 616e 672f 5374 7269 6e67 3b4a  va/lang/String;J
                00000770: 5b4c 6a61 7661 2f6c 616e 672f 5374 7269  [Ljava/lang/Stri
                00000780: 6e67 3b5b 4929 5601 0025 4c63 6f6d 2f73  ng;[I)V..%Lcom/s
                00000790: 756e 2f62 7472 6163 652f 616e 6e6f 7461  un/btrace/annota
                000007a0: 7469 6f6e 732f 4f6e 4d65 7468 6f64 3b01  tions/OnMethod;.
                000007b0: 0005 636c 617a 7a01 0012 2f2e 2a5c 2e4f  ..clazz.../.*\.O
                000007c0: 6e4d 6574 686f 6454 6573 742f 0100 066d  nMethodTest/...m
                000007d0: 6574 686f 6401 0008 6c6f 6361 7469 6f6e  ethod...location
                000007e0: 0100 254c 636f 6d2f 7375 6e2f 6274 7261  ..%Lcom/sun/btra
                000007f0: 6365 2f61 6e6e 6f74 6174 696f 6e73 2f4c  ce/annotations/L
                00000800: 6f63 6174 696f 6e3b 0100 0576 616c 7565  ocation;...value
                00000810: 0100 214c 636f 6d2f 7375 6e2f 6274 7261  ..!Lcom/sun/btra
                00000820: 6365 2f61 6e6e 6f74 6174 696f 6e73 2f4b  ce/annotations/K
                00000830: 696e 643b 0100 0545 5252 4f52 0800 2801  ind;...ERROR..(.
                00000840: 001a 636f 6d2f 7375 6e2f 6274 7261 6365  ..com/sun/btrace
                00000850: 2f42 5472 6163 6555 7469 6c73 0700 3401  /BTraceUtils..4.
                00000860: 0007 7072 696e 746c 6e01 0015 284c 6a61  ..println...(Lja
                00000870: 7661 2f6c 616e 672f 4f62 6a65 6374 3b29  va/lang/Object;)
                00000880: 560c 0036 0037 0a00 3500 3801 000d 436f  V..6.7..5.8...Co
                00000890: 6e73 7461 6e74 5661 6c75 6501 0004 436f  nstantValue...Co
                000008a0: 6465 0100 0d53 7461 636b 4d61 7054 6162  de...StackMapTab
                000008b0: 6c65 0100 1952 756e 7469 6d65 5669 7369  le...RuntimeVisi
                000008c0: 626c 6541 6e6e 6f74 6174 696f 6e73 0021  bleAnnotations.!
                000008d0: 0002 0004 0000 0002 0009 0006 0007 0000  ................
                000008e0: 0049 0008 0009 0001 003a 0000 0002 000a  .I.......:......
                000008f0: 0003 0009 000b 000c 0001 003b 0000 0046  ...........;...F
                00000900: 0006 0000 0000 0025 1202 0101 0101 01b8  .......%........
                00000910: 0014 b300 16b2 0016 b800 1a9a 0007 b800  ................
                00000920: 1db1 b800 20b1 b800 24b8 001d b100 0100  .... ...$.......
                00000930: 0000 1e00 1e00 0e00 0100 3c00 0000 0700  ..........<.....
                00000940: 021a 4307 000e 0001 0025 000c 0001 003b  ..C......%.....;
                00000950: 0000 0011 0001 0001 0000 0005 2ab7 0027  ............*..'
                00000960: b100 0000 0000 0900 2800 2900 0200 3b00  ........(.)...;.
                00000970: 0000 3b00 0100 0600 0000 1ab2 0016 b800  ..;.............
                00000980: 1a9a 0004 b112 33b8 0039 b800 1db1 b800  ......3..9......
                00000990: 24b8 001d b100 0100 0a00 1300 1300 0e00  $...............
                000009a0: 0100 3c00 0000 0700 020a 4807 000e 003d  ..<.......H....=
                000009b0: 0000 001e 0001 002a 0003 002b 7300 2c00  .......*...+s.,.
                000009c0: 2d73 0028 002e 4000 2f00 0100 3065 0031  -s.(..@./...0e.1
                000009d0: 0032 0001 003d 0000 0006 0001 0005 0000  .2...=..........
                000009e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                000009f0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a30: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a50: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000a90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000aa0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ab0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ac0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ad0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ae0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000af0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b30: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b50: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000b90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ba0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000bb0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000bc0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000bd0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000be0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000bf0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c30: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c50: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000c90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ca0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000cb0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000cc0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000cd0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ce0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000cf0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d30: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d50: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000d90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000da0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000db0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000dc0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000dd0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000de0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000df0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e30: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e50: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000e90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ea0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000eb0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ec0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ed0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ee0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ef0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f00: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f10: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f20: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f30: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f40: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f50: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f60: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f70: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f80: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000f90: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000fa0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000fb0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000fc0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000fd0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000fe0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
                00000ff0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
