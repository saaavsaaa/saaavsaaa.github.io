

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



利用魔数验证了加载的文件是class文件后，用去掉了魔数后的byte数组创建BTraceProbePersisted对象。

        BTraceProbePersisted bpp = new BTraceProbePersisted(this);
                try (DataInputStream dis = new DataInputStream(new ByteArrayInputStream(Arrays.copyOfRange(code, 4, code.length)))) {
                    bpp.read(dis);
                    return bpp;


read

javap -verbose +<class>
