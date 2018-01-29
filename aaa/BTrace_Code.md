

读入Btrace自己编译的脚本class文件
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

magic:class是一组以8位字节为基础单位的二进制流。在class文件开头的四个字节， 存放着class文件的魔数， 这个魔数是class文件的标志，他是一个固定的值： 0XCAFEBABE 。 也就是说他是判断一个文件是不是class格式的文件的标准， 如果开头四个字节不是0XCAFEBABE， 那么就说明它不是class文件， 不能被JVM识别。（可参考《深入理解Java虚拟机JVM高级特性与最佳实践》第六章）

这句代码是要将4个字节合成一个值与BTraceProbePersisted.MAGIC值进行比较。因为byte转为int，int会用32位中的低8位保存这个数，所以前3个字节分别进行了左位移。其中对每个字节和0xff的位与运算是因为：计算机中都是用的补码，因为计算机只能做加减运算，为了在运算中保持符号位的正确性所以使用的是补码。而这条语句一旦开始执行code[0]就会被表示为int类型，变为32位的，如果不做处理那么高的24位会补1导致补码值不正确，而先做&0xff运算，原来是0的还是0，是1的还是1，就保证了数据的正确性。(每个字节表示数的取值范围：因为-0的符号没有意义,所以符号位是1，其余都是0的补码可以用来表示-127-1，因为负数的补码符号位为1，其余位为该数绝对值的原码按位取反，然后加1)。code如果是我们自己编译的class文件读出来的byte数组，code[0] code[1] code[2] code[3]指的就是magic这4个字节。为什么要这么说呢，因为如果我们传给btrace的是个.java文件，它自己会进行编译，编译的class格式也是参照了虚拟机的格式，但是其中魔数用的是0xbacecaca，版本号用的是VERSION = 1，见BTraceProbePersisted.write(DataOutputStream dos)方法。

        $ xxd ArgsDurationErrAAA.class 
        00000000: bace caca 0000 0001 0022 7472 6163 6573 
        00000010: 2f6f 6e6d 6574 686f 642f 4172 6773 4475
        00000020: 7261 7469 6f6e 4572 7241 4141 0000 0000
        00000030: 0000
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

readUTF方法先读取流的下两个字节，转为一个无符号的16位整数，对应BTraceProbePersisted.write中写版本后的dos.writeUTF(getClassName(true))，上面xxd命令已经显示了这两个字节是0022, 2 * 16 + 2 = 34。读出来的值utflen为34。在不小于utflen的情况下DataInputStream使用80个二进制位，也就是40个字节存储类名信息，如果小于utflen则使用utflen的两倍。这里值是34，所有会有6位多了出来，上面xxd结果后面的三组0000



        while (count < utflen) {
            c = (int) bytearr[count] & 0xff;
            if (c > 127) break;
            count++;
            chararr[chararr_count++]=(char)c;
        }

如果有出现c > 127的情况，下面还有一个循环处理。最后转成字符串返回，就是类全路径名了。

        private void readServices(DataInputStream dis) throws IOException {
                int num = dis.readInt();
                for (int i = 0; i < num; i++) {
                    delegate.addServiceField(dis.readUTF(), dis.readUTF());
                }
        }

readInt读入下面4个字节，读取逻辑和刚才是一样的，delegate.addServiceField：

                void addServiceField(String fldName, String svcType) {
                        serviceFields.put(fldName, svcType);
                }

readOnMethods:

            private void readOnMethods(DataInputStream dis) throws IOException {
                int num = dis.readInt();
                for (int i = 0; i < num; i++) {
                    OnMethod om = new OnMethod();

接着前面方法后的4个自己标识有多少个方法，依次处理每个方法：

            om.setClazz(dis.readUTF());                   设置被拦截的类，例如：/.*\.OnMethodTest/

            om.setMethod(dis.readUTF());                  被拦截的方法，例如：args；会判断是否正则，是否是注解

            om.setExactTypeMatch(dis.readBoolean());      下一个byte转为boolean
            
            bom.setTargetDescriptor(dis.readUTF());
            om.setTargetName(dis.readUTF());
            om.setType(dis.readUTF());
            om.setClassNameParameter(dis.readInt());
            om.setDurationParameter(dis.readInt());
            om.setMethodParameter(dis.readInt());
            om.setReturnParameter(dis.readInt());
            om.setSelfParameter(dis.readInt());
            om.setTargetInstanceParameter(dis.readInt());
            om.setTargetMethodOrFieldParameter(dis.readInt());
            om.setMethodFqn(dis.readBoolean());
            om.setTargetMethodOrFieldFqn(dis.readBoolean());
            om.setSamplerKind(Sampled.Sampler.valueOf(dis.readUTF()));
            om.setSamplerMean(dis.readInt());
            om.setLevel(dis.readBoolean() ? Level.fromString(dis.readUTF()) : null);
            Location loc = new Location();
            loc.setValue(Kind.valueOf(dis.readUTF()));
            loc.setWhere(Where.valueOf(dis.readUTF()));
            loc.setClazz(dis.readBoolean() ? dis.readUTF() : null);
            loc.setField(dis.readBoolean() ? dis.readUTF() : null);
            loc.setMethod(dis.readBoolean() ? dis.readUTF() : null);
            loc.setType(dis.readBoolean() ? dis.readUTF() : null);
            loc.setLine(dis.readInt());
            om.setLocation(loc);
            delegate.addOnMethod(om);
